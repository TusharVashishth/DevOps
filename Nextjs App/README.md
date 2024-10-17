## In this tutorial we are going to use AWS Lightsail to host our Next js App

**Note You can use any other VPS service provider like DigitalOcean**

During our LightSail Machine creation we have to generate the SSH-KEY on our machine.

So we can use this command to generate SSH-KEY on our machine

```bash
ssh-keygen
```
After hitting this command It will ask you to enter the name for your **SSH-KEY** So we will enter **aws_lightsail** as our **SSH-KEY** name.

After entering the name just click **Enter** Do not pass any password.

It will generate 2 files one is public key and one is private key

#### Let's Connect our server by our terminal using SSH-KEY

```bash
ssh -i private-key username@YOUR-IP
```
**After Successfully Login.First we will update our Linux Machine.**

```bash
sudo apt update
sudo apt upgrade -y
```
**After Updating the system now let's install Nodejs**

```bash
sudo apt-get install npm
```

After hitting above command run below command to check node version
```bash
node -v
```
After this command you probably see old **Node js** version.
So to install new Node js version we have to install **package n** globally in our system

```bash
npm i -g n
```
After installing the **n package** now you can install any node js version . if you want to install any particular version then you can install that like this

```bash
n 20.13.1
```

Or if you want to install latest **LTS version** then you can install that like this
```bash
n lts
```
After installing node js version you have to exit from terminal and relogin to see the updated node js version.

**Now Let's install PM2**

```bash
sudo npm install -g pm2
```

After installing **PM2** we have to HIT one more command. Which automatically restart PM2 whenever our system reboot.

```bash
pm2 startup systemd
```

### Now Let's Install Nginx on our machine

```bash
sudo apt-get install nginx -y
```

### After installing Nginx now let's clone our Next js project and after cloning run below commands

```bash
npm install
npm run build
```

### After build command let's start our app using PM2
```bash
pm2 start npm --name "YOUR-APP-NAME" -- start
pm2 save
```

**Note** When we install Nginx then it's give us default configuration file and you can check that file by going below path.

```bash
cd /etc/nginx/sites-available
sudo nano default
```
but we will not use this default file we will create our new **conf** file. So let's unlink default file.

```bash
sudo unlink /etc/nginx/sites-enabled/default
```
and now let's create our new conf file inside sites-available directory.

```bash
sudo nano /etc/nginx/sites-available/YOUR-DOMAIN.conf
```
Right now we have only one app So we will add only one Server Block
```bash
server {
    listen 80;
    server_name DOMAIN;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

and after this we will create a Symbolic link for our new conf file
```bash
sudo ln -s /etc/nginx/sites-available/YOUR-DOMAIN.conf /etc/nginx/sites-enabled/
```
**After these all changes we have to reload our nginx server**

```bash
sudo nginx -t # To check if all syntax is okay
sudo systemctl reload nginx
```
Now when you open your public IP on browser tab you should see your next js app.

**Now let's add SSL on our domain using certbot**
```bash
sudo apt install certbot python3-certbot-nginx -y
```
```bash
sudo certbot --nginx -d your_domain -d www.your_domain
```

**Note-** After this you may see your sites stopped working. So for this we have to check our network fireall that 443 PORT is allow or not.

And that's it now your site deployed successfully!

**Bonus commands**

To check Nginx error logs
```bash
sudo tail -f /var/log/nginx/error.log
```

Certbot automatically sets up a cron job to renew SSL certificates. You can manually test this with:
```bash
sudo certbot renew --dry-run
```

to allow all PORT for firewall

```bash
sudo ufw allow 'Nginx Full'
sudo ufw reload
```
**Sometime you may get PORT Error. So to check if application on PORT is running or not you can use below command**

```bash
sudo ss -nlp | grep 3000 #If this not work
sudo lsof -i :3000 # You can also try this command
```

After hitting any above command if you see any application running then you can kill it .

```bash
sudo kill -9 1234 #this last number is PID it may be different for you
```