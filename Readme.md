# Partie I : CICD du site web statique 

## Prérequis
- Docker
- Ansible
- Jenkins

## Contexte et objectifs
Vous disposez d'un site web statique [disponible à cette adresse](https://github.com/diranetafen/static-website-example.git)

Le but de ce projet est de mettre en place un pipeline complet cicd sur votre application. Dans un premier temps, il faudra conteneuriser l'application.
Pour celà, quelques intructions pour vous orienter : 

- un squelette de **Dockerfile**  vous est fournie, il faut le compléter.
    Il faudrait refaire exactement les même étapes que celles dans le cours, à quelques différences prêt : 
- Pour lancer le conteneur après le build, voici un exemple de commande : 
 ```
docker run --name cont_name -d -e PORT=80 -p 80:80 $ID_DOCKERUB/$IMAGE_NAME:$IMAGE_TAG
 ```

- Le déploiement en **staging** et **production** sera fait sur des VMs et non dans Heroku
- On utilisera **Ansible** pour se faciliter la vie
    - Qui dit ansible dit créer des **playbooks** dans le repos github

- Pour vous aider, un vagranfile vous est fourni, ce dernier déploie : 
    - Le **serveur jenkins**, avec **docker** et **ansible** installé sur ce dernier
    - Les deux serveurs de **Staging** et **de Production**, aucun outil d'installé sur ces serveurs
    - Les trois serveurs sont joignables au niveau réseau



## Aide pour la partie déploiement avec ansible uniquement
- Un exemple d'inventaire, **ansible.cfg**, **host_vars** et **group_vars** sont fournis, vous pouvez vous en inspirer...
- Le playbook d'installation de docker (**docker.yml**) est fourni dans le dossier **ressources ansible**
- Un utilisateur **jenkins** existe sur les 3 machines crées, il sera utilisé comme user de connexion par ansible
    - Le répertoire personnel du user **jenkins** est **/var/lib/jenkins**  sur les 3 serveurs
    - Cet utilisateur **jenkins** ne possède pas de mot de passe, mais une paire de clés ssh.
        - Le couple de clés se trouve donc dans **/var/lib/jenkins/.ssh**
        - La clés publique est disponible sur les serveurs worker1 et worker2

        - Pour passer la clés dans les commandes ansible, utiliser l'option : ```--private-key <fichier de la clés privée>```
          * Example :  ``` ansible all -m ping --private-key /var/lib/jenkins/.ssh/id_rsa```
    - la clés privée pourr être rajoutée comme secret dans le serveur jenkins, et être récupérée dans une variable
        * Exemple : ```PRIVATE_KEY = credentials('private_key')```
    - Cette clé privée  pourrait être passée dans les commandes ansible pour se connecter aux machines, vous pouvez la récupéer dans votre projet si vous le souhaiter (c'est pas obloigatoire), comme suit : 
        ```
        echo $PRIVATE_KEY > id_rsa
        chmod 600 id_rsa
        ```

# PARTIE II : Wordpress As Code avec ANSIBLE
L'entreprise POZOS souhaite metre en place on site web vitrine.
Elle fait appel à vos services afin de mettre en place le site à travers le CMS Wordpress. 

## Architecture de Wordress
L'application wordpress se découpe en deux grand modules 
- Le frontend qui est la partie Vitrine. Généralement, on utilise du **apache/php**
- Le backend qui est la partie Base de Donnée. On utilise généralement de **mysql**  ou **mariadb**

POZOS vous recommande d'utiliser du **apache - php - mysql/mariadb**, car ils sont très utilisés sur leurs infrastructures de prod.
Ca tombe bien, la stack **LAMP** (***Linux, Apache, Mysql/Mariadb, PHP***) serait idéale afin de répondre à cette problématique.

POZOS souhaite profiter de ce projet pour adopter les bonnes pratiques **agiles/DevOps**, notamment l'**Insfrastructure as Code (IaC)**.
Celà leur permettra de péréniser votre travail et de pouvoir versionner leurs infrastructure.

Etant donné une procédure d'installation de wordpress,votre mission, si vous l'accepter est **d'automatiser le déploiement de wordpress à l'aide de Ansible**, 


## Procédue d'installation de wordpress 

#### Prérequis
- OS : Centos 7
- Privilèges administrateurs sur la machine cible


#### Installation de la stack LAMP
1. ***Installation et configuration du mysql/mariadb***
   1. Installation de mariadb : 
        ```
        [vagrant@client1 ~]$ sudo yum install -y mariadb-server
        ```
    2. Connexion à la base de donnée  et creation du schema BDD + user wordpress
        > :warning: Initialement, l'utilisateur **root** de la base de donnée **ne possède pas de mot de passe**, à vous de définir son mot de passe.

        Pour se connecter à la base de donnée et la configurer, taper la suide de commandes suivantes : 
        ```
        [vagrant@client1 ~]$ mysql -h  localhost -u root
        MariaDB [(none)]> create database wordpress;
        MariaDB [(none)]> CREATE USER 'wordpress'@'localhost' IDENTIFIED BY 'wordpress';
        MariaDB [(none)]> use wordpress;
        MariaDB [wordpress]> GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost';        
        MariaDB [myexample]>EXIT
        ```

2. ***Installation du serveur web **apache*****
    ```
    sudo yum install -y httpd 
    ```
3. ***Installation de **PHP*****
   1. Installation des repos pour php7 + mise à jours du système: 
        ```
        sudo yum install -y http://rpms.remirepo.net/enterprise/remi-release-7.rpm
        sudo yum update -y
        ```
    1. Installation de php7 et ses extensions 
        ```
        sudo yum -y --enablerepo=remi-php74 install php php-bz2 php-mysql php-curl php-gd php-intl php-common php-mbstring php-xml
        sudo sudo systemctl restart httpd
        ```

4. ***Installation de Wordpres***
     ```
     sudo yum install -y wget
     wget http://WordPress.org/latest.tar.gz
     sudo tar -xzvf latest.tar.gz
     sudo mv wordpress/* /var/www/html/
     sudo chown -R apache.apache /var/www/html/
     ```

5. ***Démarrage des services apache(httpd) et mysql/mariadb***
     ```
     sudo systemctl start httpd
     sudo systemctl start mariadb
     sudo systemctl enable httpd
     sudo systemctl enable mariadb
     ```
6. ***Finalisation de l'installation dans le navigateur:***
   Pour celà, il faudrait aller dans votre navigateur et saisir l'une  des adresses suivantes : 
   - http://**<votre_ip_machine/wordpress>** 
   - http://**<votre_ip_machine>**

##### Liens Annexes : 
- [Exemple de procédure d'install de wordpress sur Centos 7](https://www.vultr.com/docs/how-to-install-wordpress-on-centos-7/#:~:text=To%20install%20WordPress%2C%20you%20need,from%20WordPress.org%20using%20wget.&text=Use%20wget%20to%20download%20the%20latest%20WordPress%20version.&text=Unzip%20the%20downloaded%20WordPress%20tar%20archive.&text=Now%2C%20move%20the%20extracted%20file,%2Fvar%2Fwww%2Fhtml%20.)
