1. Make a space on the Digital Ocean site

2. Use the API menu entry in your DigitalOcean account to create (or get) API credentials for your new space

3. Spin up the lab basic droplet 

4. Login to droplet

5. vim *.passwd-s3fs* in your home directory

   1. ```
      digitalOceanSpaceName:key:secret
      ```

6. chmod 0600 .passwd-s3fs 

7. sudo apt-get install s3fs

8. make a mount directory and mount the space 

9. ```
   mkdir /space
   s3fs digitalOceanSpaceName /your/mount/path -ourl=https://nyc3.digitaloceanspaces.com -ouse_cache=/tmp
   ```

10. get fastqs to local disk (via wget)

11. set up rclone

    1. ```
       curl https://rclone.org/install.sh | sudo bash
       ```

12. configure rclone: see https://rclone.org/s3/#digitalocean-spaces

13. use rclone to sync folders

    1. ```
       rclone copy /path/to/files spaces:name-of-space/folder
       ```