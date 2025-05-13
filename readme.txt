🔁 1. Intégration Bitbucket → AWX
🎯 Objectif :
Bitbucket (via Pipelines) déclenche automatiquement un job AWX.

⚙️ Comment faire :
Tu configures un pipeline dans .bitbucket-pipelines.yml.

Ce pipeline fait un appel API vers AWX :

bash
Copier
Modifier
curl -X POST https://awx.yourcompany.com/api/v2/job_templates/42/launch/ \
  -H "Authorization: Bearer $AWX_TOKEN"
🧩 Ce que fait AWX :
Lance un playbook Ansible.

Ce playbook va gérer :

la récupération des secrets,

la connexion SSH,

et l’exécution des scripts buildfed.py et deployfed.sh.

🔐 2. Intégration AWX → Vault (Vault Access)
🎯 Objectif :
AWX va récupérer dynamiquement les secrets nécessaires pour exécuter les scripts (ex: credentials API Gateway, certificats, .env).

⚙️ Comment faire :
Dans AWX, tu définis un Credential Type Vault.

Ton playbook inclut une tâche de ce type :

yaml
Copier
Modifier
- name: Get secrets from Vault
  ansible.builtin.uri:
    url: "https://vault.yourcompany.com/v1/secret/data/gateway"
    method: GET
    headers:
      X-Vault-Token: "{{ vault_token }}"
    return_content: yes
Tu peux aussi utiliser le plugin hashicorp.vault natif dans Ansible.

🔐 3. Intégration AWX → SSH (SSH Connection)
🎯 Objectif :
Se connecter à une VM d’exécution pour lancer le script de déploiement.

⚙️ Comment faire :
AWX utilise une clé SSH stockée dans un Credential.

Le playbook se connecte comme ceci :

yaml
Copier
Modifier
- name: Run deployfed.sh
  ansible.builtin.shell: "./deployfed.sh"
  args:
    chdir: /home/deploy/scripts
La connexion SSH est faite automatiquement via l’inventaire de la VM cible (hostname + user + clé).

🧱 Récapitulatif de l’intégration
ÉTAPE	ACTION AUTOMATISÉE
Bitbucket push	Active un pipeline CI
Pipeline CI Bitbucket	Fait un appel API vers AWX
AWX job template	Exécute un playbook Ansible
Playbook AWX	1. Récupère les secrets via Vault
2. Se connecte en SSH
Sur la VM cible	Exécute buildfed.py puis deployfed.sh
API Gateway	Reçoit le .fed via CLI ou REST API
