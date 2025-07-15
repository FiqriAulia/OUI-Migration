open webui migration



yang harus dipindahkan



1. Docker Container
2. Mount file



https://youtu.be/QskmB4fb-uo?si=sWTQPROMhJvWvaGq



1. Docker Container



sebelum melakukan step ini disarankan stop container yang akan di migrasi



untuk memindahkan docker container ke vm baru bisa menggunakan commit untuk membuat image baru yang akan di run di vm baru



sudo docker commit <container-name> <nama-image>



untuk image yang sudah atau pernah dibuat dapat dilihat di



sudo docker images



untuk memindahkan images kita perlu save image ke .tar



sudo docker save <image-name> > <backupname>.tar



setelah semua images di save dan di compress ke .tar.gz. kita bisa memindahkan file ke vm baru



2\. Mount file



lokasi mount file dapat dilihat di docker inspect. setelah mengetahui lokasi (disini berada pada /mnt/sdb1/) data/ maka bisa compress file data/ dan pindahkan ke vm baru



untuk mempermudah letak file di /mnt/sdb1/ setelah berhasil di extract akan menjadi /mnt/sdb1/data



3\. VM Baru



setelah memindahkan mount file dan docker images, sekarang akan kita lakukan setup docker container untuk berjalan di vm baru.



extract file images dan lakukan docker load



sudo docker load -I <filename>.tar



setelah berhasil akan ada nama backup imagesnya di docker images.



lakukan sudo docker run untuk open-webui, tika, dan litellm.



sudo docker run -d --name litellm   --network my-network   --ip 172.18.0.4   -p 4000:4000   -v /home/ali.ns/litellm\_config.yaml:/app/config.yaml   litellmbackup:latest



sudo docker run -d --name open-webui   --network my-network   --ip 172.18.0.3   -p 3001:8080   -v /mnt/sdb1/data/anythingllm1:/app/backend/data   ouibackup:latest



sudo docker run -d --name tika   --network my-network   --ip 172.18.0.2   -p 9998:9998   tikabackup:latest





