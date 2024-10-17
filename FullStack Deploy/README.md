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

### Now Let's open our default nginx file
```bash
cd /etc/nginx/sites-available
sudo nano default
```

### Now we have to change our location object with this
```bash
location / {
	proxy_pass http://127.0.0.1:3000; 
	proxy_http_version 1.1;
	proxy_set_header Connection "upgrade";
	proxy_set_header Host $host;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header X-Real_IP $remote_addr;
	proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
}
```
**After this we also have to pass our root path in root variable**

```bash
root /var/www/YOUR_FOLDER
```
**After these all changes we have to reload our nginx server**

```bash
sudo nginx -t # To check if all syntax is okay
sudo systemctl reload nginx
```
Now when you open your public IP on browser tab you should see your next js app.

**Now if you want to add your domain on your app then you have to add your public IP to your DNS record to point out the domain**

Now let's open again our default nginx file and add our domain.

```bash
server_name YOUR-DOMAIN
```
And again check syntax and restart the nginx
```bash
sudo nginx -t # To check if all syntax is okay
sudo systemctl reload nginx
```

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

Certbot automatically sets up a cron job to renew SSL certificates. You can manually test this with:
```bash
sudo certbot renew --dry-run
```

to allow all PORT for firewall

```bash
sudo ufw allow 'Nginx Full'
sudo ufw reload
```