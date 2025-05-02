# nodejs-uvt

a new modification was required 

To  clone this repo :

`git clone https://github.com/bogdanobogeanu/nodejs-uvt.git`

Then change folder to nodejs-uvt:

`cd nodejs-uvt`

To build the nodeJs application with Docker :

`docker build -t mynodejs:1.0.0 .`

To run the nodeJs application with Docker as a daemon :

`docker run --name mynodejs -d -p 8080:8080 -it mynodejs:1.0.0`

To check that application is working :

`curl localhost:8080/nodejs` 

To check the running containers :

`docker ps -a`

To stop the container 

`docker stop mynodejs`

To remove the container 

`docker rm mynodejs`

-----------------------------------------------------------------
# Jenkins on Rocky Linux 9
-----------------------------------------------------------------

Ce este Jenkins?
Jenkins este un automation server bazat pe Java cu care putem defini continuous integration si continuous delivery (CI/CD) jobs sau pipelines.
Continuous integration (CI) este o practica DevOps in care membrii unei echipe commit modificari ale codului regulat in GIT, dupa care porneste un build si ruleaza teste automate. Continuous delivery (CD) este tot o practica DevOps prin care modificarile codului sunt deploy-ate in productie.

Cu ce ne va ajuta pe noi Jenkins?
Vom automatiza procesul de build si deploy a imaginilor docker. Buildul se va face de catre Jenkins ori de cate ori se va comite o modificare a codului in GitHub. Apoi noua imagine se va push-ui in DockerHub automat , tot din Jeknins.

Instalare Jenkins intr-o masina virtuala care ruleaza Rocky Linux 9

Confirmați că system repositories funcționează (dupa pornirea masinii virtuale, la prima executare a comenzii sudo, o sa va ceara parola userului. In cazul meu, userul este student, la voi poate fi diferit):
sudo dnf repolist
 

Pasul 1: Instalați OpenJDK 21 pe Rocky Linux 9
Listați versiunile disponibile ale JDK pe Rocky Linux 9:
sudo dnf search java-*-openjdk
 
O să instalăm pachetul java-21-openjdk:
sudo dnf -y install java-21-openjdk
 
La finalul instalarii o sa vedem mesajul Complete! Si cateva detalii despre pachetele instalate, java si dependintele necesare:
 

Verificam versiunea de java instalata default pe masina noastra virtuala:
java -version
 
Pasul 2: Adăugați repository-ul Jenkins la Rocky Linux
Echipa Jenkins menține un repository cu pachete RPM pentru Jenkins. Vom adăuga acest repository, apoi vom instala pachete din el.
Utilizați comanda wget pentru a descărca fișierul jenkins.repo și plasați-l în directorul corect:
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
 
De asemenea, importați cheia GPG folosită pentru a semna pachetele Jenkins:
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
 
Verificam daca repository-ul Jenkins este disponibil local:
sudo dnf repolist
 
Putem vedea pe ultima linie, coloana intai, repo id numit “jenkins”.

Step 3: Install Jenkins Server on Rocky Linux 9
Comanda de instalare a lui Jenkins:
sudo dnf -y install jenkins
 

Porniți serviciul Jenkins pe Rocky Linux 9 (dupa ce rulati comanda de start, asteptati in jur de 1 minut ca Jenkins sa porneasca):
sudo systemctl start jenkins
 
Trebuie sa configuram Jenkins sa porneasca odata cu masina virtuala (system boot):
sudo systemctl enable jenkins
 
Verificam daca Jenkins intr-adevar ruleaza pe masina noastra virtuala:
 
Putem observa cuvintele de culoare verde: active (running). Asta ne spune ca Jenkins este pornit. Daca din anumite motive nu va porni, o sa vedeti un mesaj de culoarea rosie care va incepe probabil cu cuvantul “fail” sau “failed”.

Pasul 4: Configurați Jenkins pe Rocky Linux 9 din interfața web
După instalare și pornirea serviciului, mergeți la consola browserului web pe URL:
http://localhost:8080
Ar trebui să vedeti o pagină de bun venit și instructiuni despre cum să obțineți parola inițială de administrator:
 
Ca sa vedem parola initiala, trebuie sa efectuam comanda cat pe fisierul /var/lib/jenkins/secrets/initialAdminPassword (Parola initiala e diferita la fiecare instalare, cand instalati voi, o sa aveti o alta parola initiala):
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
 
Copy si paste la parola in pagina Jenkins in casuta Administrator Password si apoi apasati butonul Continue:
 
Dupa ce ati adaugat parola, O sa apara fereastra urmatoare, unde selectati “Select plugins to install”.
  

La urmatorul pas, cautam dupa “git” si o sa instalam doar pluginul “Git”. O sa mai apara si alte pluginuri, asa ca verificati cu atentie sa aiba simplu, numele Git, ca in poza urmatoare:
 
 
Asteptam sa termine de instalat toate pluginurile necesare.

Dupa acest pas, o sa ne configuram userul de administrare si apasam butonul Save and Continue:
 













Dupa acest pas, o sa apara pasul de Instance Configuration, nu modificam nimic, doar apasam pe Save and Finish.
 
Apasam pe butonul de Start using Jenkins:
 
Am setat Jenkins si putem sa incepem sa il folosim. Urmatoarea poza ne arata prima pagina care apare dupa ce am terminat setup-ul. 
 

In acest moment al instalarii de Jenkins, consideram ca aveti deja instalat pe masina virtuala, “podman” (identic cu docker, ar trebui sa il aveti instalat de la cursul de podman) si consideram ca acesta este setat sa porneasca la startup. Putem folosi comanda docker sau comanda podman, depinde de preferinta.
Consideram ca aveti instalat pe masina virtuala, “git” (ar trebui sa il aveti instalat de la cursul de git).
Mai consideram ca aveti cont pe Docker Hub https://hub.docker.com/
O sa avem nevoie de acest cont pentru a face push imaginii careia ii vom face build din Jenkins in Docker Hub.
Consideram ca ati creat un repository in DockerHub cu numele “mynodejs” pentru a putea face push imaginilor create cu Jenkins:
Selectati Create repository in pagina web a DockerHub:
 


Apoi adaugati “mynodejs” la Repository Name si apasati Create, la fel ca in imaginea de mai jos:
 

Pasi necesari pentru a pregati Jenkins pentru a face push sau upload imaginii careia ii vom face build

Executati comanda urmatoare:
sudo chmod 666 /var/run/docker.sock
 
Comanda “chmod 666 /var/run/docker.sock” trebuie rulata de fiecare data cand pornim masina virtuala, pentru ca fisierul docker.sock este fisier nou de fiecare data cand porneste docker si isi va pierde permisiunile cand porniti masina virtuala si implicit si docker porneste din nou. 
Pentru a rezolva acesta problema, vom seta un cron job care la pornirea masinii virtuale sa ruleze comenzile “sleep 60 && chmod 666 /var/run/docker.sock”, sa nu trebuiasca sa le rulam noi manual de fiecare data. Am adaugat si comanda sleep 60 care inseamna ca, inainte sa ruleze comanda chmod, va astepta 60 de secunde. De ce? Pentru ca e posibil ca laptopurile voastre personale sa nu fie foarte performante si sa fim siguri ca docker a pornit inainte sa rulam comanda.
Pentru aceasta, trebuie sa folosim userul “root”. Ne logam ca root cu comanda de mai jos, introducem parola userului nostrum (daca nu am introdus recent parola la una din comenzile rulate cu “sudo”) si am devenit root, asa cum se vede in poza:
sudo su –
 
Rulam comanda “crontab -e” pentru a adauga un cron job:
 
Adaugam aceasta linie in fisierul care se deschide:
@reboot sleep 60 && chmod 666 /var/run/docker.sock
 
Salvam fisierul si apoi putem sa verificam daca am adaugat cum trebuie cron job-ul cu comanda
crontab -l
  
Acum putem continua tutorialul, efectuand comanda “exit” ca sa revenim la userul nostru, in cazul meu “student”:
 

In situatia in care nu reusiti sa setati acest cron job de mai sus, puteti rula manual, dupa fiecare pornire a masinii virtuale, comanda “sudo chmod 666 /var/run/docker.sock” cu userul vostru normal, nu neaparat cu root.




Continuam tutorialul - Pentru a putea executa comezi ca si userul jenkins:
sudo usermod -s /usr/bin/bash jenkins
sudo su - jenkins
 
 
Urmatorul pas este sa facem login in docker hub cu userul nostru personal pe care l-am facut deja in Docker Hub. Comanda generica este aceasta, insa in poza o sa vedeti ca apare userul meu. La voi, o sa trebuiasca sa folositi userul vostru creat in Docker Hub si apoi tastati parola setata de voi.
docker login -u <username>
or 
podman login -u <username>
 
Dupa login, tastam comanda exit sa revenim la shell-ul userului student:
 
Dupa cum obervam, comanda de login in docker hub, de mai sus are un WARN:
  
Acest WARN o sa impacteze viitoarele noastre build-uri cu docker, asa ca o sa executam ca root comanda care apare in WARN ca sa nu avem erori la build-ul imaginilor de docker in Jenkins:
sudo su -
loginctl enable-linger 981
 
Va rog sa fiti atenti la numarul din WARN, in cazul nostrum 981. Acesta este id-ul user-ului Jenkins si poate fi diferit in cazul vostru. Daca apare alt numar, executati comanda exact cum apare la voi in WARN.

As root user, we need to run next two commands to fix other problems on buid docker images with Jenkins:
echo Jenkins:10000:65536 >> /etc/subuid
echo Jenkins:10000:65536 >> /etc/subgid
 

Jenkins automation
Exercitiu:
Creati un nou Job in Jenkins care sa faca build unei imagini de docker, sa ii puna tag-uri si sa ii faca push in docker hub.
Pasul 1.
Adaugare credentiale username si parola pentru Docker Hub

Dupa ce am pregatit job-ul, trebuie sa adaugam user is parola de docker hub ca si variabile in Jenkins. Pentru aceasta, trebuie sa mergem la Manage Jenkins:
 
Click on Credentials
 
Click on System
 
Click on Global credentials (unrestricted)
 
Click on blue button on the right Add Credentials
 





Aici adaugam credentials ca in poza urmatoare:
Lasam Kind asam cum este – Username with password.
Adaugam propriul username pentru docker hub, propria parola si ID numit DOCKER_CREDENTIALS si descrierea “Docker username and password”:
 
Click on Create.

Pasul 2.
Mergeti in pagina default a lui Jenkins si selectati “New item”
 

Adaugati un nume, selectati “Freestyle project” si apasati Ok
 











La pasul General, adaugati o descriere, apoi scroll la pasul urmator
 

La pasul Source Code Management selectati Git la Repository URL si adaugati https://github.com/bogdanobogeanu/nodejs-uvt.git
 

Mai jos, la acelasi pas, avem setarea Branches to build, lasam default, master
 

La pasul Triggers Selectati Pool SCM si la Schedule adaugati:  H/5 * * * * . Acest schedule o sa verifice in git la fiecare 5 minute daca exista o schimbare in cod si o sa porneasca job-ul creat de noi.
 

La pasul Environment bifam casuta cu Use secret text(s) or file(s) apasam pe Add si selectam Username and password (separated):
 
Apoi adaugam variabilele ca in poza de mai jos – Username Variable: DOCKER_USER si Password Variable: DOCKER_PASSWORD. Credentials se vor selecta automat pentru ca avem un singur credentials adaugat, cel de Docker username and password:
 
La pasul Build Steps selectam Add build step si apoi Execute Shell
 
Apoi adaugam in fereastra comanda de build: 
docker build -t mynodejs:1.0.0 .
  

Mai adaugam inca un Build step si apoi Execute Shell si adaugam comenzile pentru a porni si testa imaginea careia i-am facut build la pasul anterior
docker run –name mynodejs -d -p 8081:8080 -it mynodejs:1.0.0
sleep 10
curl localhost:8081/nodejs
 

Adaugam inca un Build step si inca un Execute shell unde rulam comenzile de stop al containerului mynodejs si apoi comanda de delete a containerului mynodejs
docker stop mynodejs
docker rm mynodejs
 



Adaugam un ultim Build step si un Execute shell pentru face login in Docker Hub si pentru a seta tag-urile imaginii construite de Jenkins. Cu ajutorul tagurilor setate correct, vom putea face push la imaginea creata in Docker Hub. Jenkins server are un set de variabile predefinite, printre care si BUILD_NUMBER , pe care il vom folosi pentru a face tag-ul imaginii docker :

docker login -u $DOCKER_USER -p $DOCKER_PASSWORD docker.io 

docker tag mynodejs:1.0.0 bogdan1980b/mynodejs:latest

docker push bogdan1980b/mynodejs:latest

docker tag mynodejs:1.0.0 bogdan1980b/mynodejs:${BUILD_NUMBER}

docker push bogdan1980b/mynodejs:${BUILD_NUMBER}

Inlocuiti userul bogdan1980b din comenzi cu userul personal de DockerHub.
 

Selectati apoi Save si apoi Build now. O sa vedeti in stanga, jos, build-ul caruia i-am dat start.

 
Este posibil ca jobul pornit sa dea fail din cauza unui bug de sistem
Daca toti pasii au fost efectuati correct, o sa vedem ca job-ul nostru a fost finalizat cu success:
 
In caz ca apar erori, putem accesa Output-ul consolei job-ului unde vom putea vedea logurile. Trebuie doar sa selectam job-ul din stanga jos  si apoi Console Output
 
 

Verificati apoi in contul vostru de DockerHub ca aveti ultima versiune de imagine.
