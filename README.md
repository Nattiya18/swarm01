# Ref
- [https://github.com/docker/awesome-compose/tree/master/plex]
# Wakatime project
- [https://wakatime.com/@spcn16/projects/clxvdqcsje]

# Create VM template
*  Ubuntu 22.04
*  CPU 2 cores
*  Memory 2024 MB
*  Storege 32 GB
# Install Docker

     sudo -i
* Set timezone Bangkok Thailand

       timedatectl set-timezone Asia/Bangkok
* Install docker engine for Ubuntu

           apt update ; apt upgrade -y
           apt-get install \
               ca-certificates \
               curl wget \
               gnupg \
               lsb-release -y
               
               mkdir -m 0755 -p /etc/apt/keyrings
               curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
               
               echo \
               "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
               $(lsb_release -cs) stable" |  tee /etc/apt/sources.list.d/docker.list > /dev/null
               
               apt-get update
               apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
       
 # Install Nala
           wget https://gitlab.com/volian/nala/uploads/605d833bdffd23cee4bb6670b2d6c27b/nala_0.12.1_all.deb
           dpkg -i nala_0.12.1_all.deb 
           apt-get -f install -y  
           
           
           tee -a /etc/apt/sources.list.d/nala-sources.list <<EOF 
          # Sources file built for nala
          deb https://mirror1.ku.ac.th/ubuntu/ jammy main restricted universe multiverse
          deb https://mirrors.nipa.cloud/ubuntu/ jammy main restricted universe multiverse
          deb https://mirror.kku.ac.th/ubuntu/ jammy main restricted universe multiverse
          deb http://mirror1.totbb.net/ubuntu/ jammy main restricted universe multiverse
          EOF
          
          nala update  
          nala upgrade -y 
          nala list —upgradable
          nala install htop dnsutils mtr -y
* reboot node

          reboot
### Clone VM 3 ตัว คือ manager work1 และ work2

          cp /dev/null /etc/machine-id
          rm /var/lib/dbus/machine-id
          ln -s /etc/machine-id /var/lib/dbus/machine-id
          init 0

# Swarm init
           docker swarm init
* run token บน work1 และ work2

            docker node ls
            
# Install portainer for swarm
            curl -L https://downloads.portainer.io/ce2-17/portainer-agent-stack.yml -o portainer-agent-stack.yml  #โหลด portainer จากลิ้งค์ดังกล่าว 
            docker stack deploy -c portainer-agent-stack.yml portainer  #คำสั่งติดตั้ง portainer
      
# Install Traefik
* สร้างไฟล์ traefik-host.yml
* Create a network

           docker network create --driver=overlay traefik-public
           
* Create tag 

           export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
           echo $NODE_ID
* Install traefik ใน stack ใน portainer

          docker stack deploy -c traefik-host.yml traefik
          
# Install Swarmpit
* กำหนด domain

         export DOMAIN=swarmpit.cpedemo.local
     
* กำหนดค่า lable เพื่อใช้สำหรับ CouchDB database

        export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
        docker node update --label-add swarmpit.db-data=true $NODE_ID
     
* กำหนดค่า lable เพื่อใช้สำหรับ Influx database

       export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
       docker node update --label-add swarmpit.influx-data=true $NODE_ID
     
* Deploy Stack 

       docker stack deploy -c swarmpit.yml swarmpit
     
* check stack 

      docker stack ps swarmpit
      docker service logs swarmpit_app
      
* ลองทดสอบเข้า Swarmpit

      swarmpit.cpedemo.local
     
# สร้าง image สำหรับการเตรียม push ขึ้น Docker hub

          sudo docker images
       
  * Login Docker
  
          docker login
      
  * กำหนด tag
          
          docker tag linuxserver/plex:latest nattiya18/plex:plex1
  
  * Push ขึ้นไปที่ Docker Hub 
  
         docker push nattiya18/plex:plex1

![image](https://user-images.githubusercontent.com/119166253/224608019-29e50de9-4bf4-49fd-b8c0-282fade806b8.png)

  Add Stack ใน Portainer โดยใช้โค้ด
       
              version: '3.3' 
                  services:
                       web: 
               image: nattiya18/plex:plex1
               networks: 
               - webproxy 
               logging:
               driver: json-file
               container_name: plexeye
               deploy: 
               replicas: 1 
               labels: 
                   - traefik.docker.network=webproxy
                   - traefik.enable=true
                   - traefik.http.routers.plexeye-https.entrypoints=websecure
                   - traefik.http.routers.plexeye-https.rule=Host("plexeye.xops.ipv9.xyz")
                   - traefik.http.routers.plexeye-https.tls.certresolver=default
                   - traefik.http.services.plexeye.loadbalancer.server.port=32400
              resources: 
              reservations: 
              cpus: '0.1'
              memory: 10M
              limits: 
              cpus: '0.4'
              memory: 160M
              networks: 
              webproxy: 
              external: true
    
   ![image](https://user-images.githubusercontent.com/119166253/224608261-6eadfd5b-746e-40df-ae15-9788c915ef96.png)
   
   
ทดลองเข้าเพื่อดูผลลัพธ์ของ image 

![image](https://user-images.githubusercontent.com/119166253/224609171-6d60f1b3-f03f-4a15-ac4c-b378ef696170.png)
