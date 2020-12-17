###Install offical nginx roles  
* role to install nginx
`ansible-galaxy install nginxinc.nginx`
* role to upload custom configs, use to replace default that takes over port 80/443
`ansible-galaxy install nginxinc.nginx_config`
  *  setup upload.yml to replace the config files

  

Need to find roles for managing nginx
how to install?
head node already has httpd installed with many used ports ie 80 & 443
can I do a test install that uses other ports? is there a default startup on port 80?