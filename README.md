# Web Infrastructure Lab: Hosting a Portfolio Website with Nginx on WSL

## Lab Goal

Last session, we covered the theory of web servers and Nginx: what it is, why servers use it, and how it helps serve websites.

This week, you will configure Nginx in a real lab environment.

By the end of this lab, your group should have:

- Cloned an existing portfolio website into Ubuntu WSL.
- Served the portfolio website using Nginx.
- Opened the website from the Windows browser on the server laptop.
- Allowed other students in the group to access the website using the school Wi-Fi.
- Allowed other students to SSH into the server laptop from their own Windows machines.

---

## Activity Diagram 

<img width="1536" height="1024" alt="Activity" src="https://github.com/user-attachments/assets/e01d96de-b88b-45f0-8dd9-e25649773d63" />


## Group Setup

Work in groups.

Choose one student’s laptop to act as the server.

That laptop must have:

- Windows
- Ubuntu WSL installed
- Nginx installed or ready to install
- SSH already configured from the previous session
- Access to the school Wi-Fi
- Administrator access on Windows

Other group members will act as clients. They will try to:

- SSH into the server laptop.
- Access the hosted portfolio website from their own laptops.

---

## Part 1: Install and Start Nginx in Ubuntu WSL

Open Ubuntu WSL on the server laptop.

Run:

```bash
sudo apt update
sudo apt install nginx git -y
```

Start Nginx:

```bash
sudo service nginx start
```

Check that Nginx is running:

```bash
sudo service nginx status
```

You can also test it from inside WSL:

```bash
curl http://localhost
```

You should see HTML output from the default Nginx page.

---

## Part 2: Clone the Portfolio Website into `/var/www`

Go to the web server directory:

```bash
cd /var/www
```

Clone the portfolio website into a folder called `portfolio`:

```bash
sudo git clone https://github.com/bedimcode/portfolio-responsive-complete.git portfolio
```

Change ownership of the project folder so your current Ubuntu user can work with it:

```bash
sudo chown -R $USER:$USER /var/www/portfolio
```

Check that the files are there:

```bash
ls /var/www/portfolio
```

You should see files such as:

```text
index.html
assets
README.md
```

---

## Part 3: Configure Nginx to Serve the Portfolio Website

Create a new Nginx configuration file:

```bash
sudo nano /etc/nginx/sites-available/portfolio
```

Paste this configuration:

```nginx
server {
    listen 8080;
    listen [::]:8080;

    server_name _;

    root /var/www/portfolio;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Save and exit.

In Nano:

```text
CTRL + O
Enter
CTRL + X
```

Enable the new website:

```bash
sudo ln -sf /etc/nginx/sites-available/portfolio /etc/nginx/sites-enabled/portfolio
```

Remove the default Nginx site:

```bash
sudo rm -f /etc/nginx/sites-enabled/default
```

Test the Nginx configuration:

```bash
sudo nginx -t
```

If the test is successful, reload Nginx:

```bash
sudo service nginx reload
```

---

## Part 4: Test the Website on the Server Laptop

On the same Windows laptop, open a browser and go to:

```text
http://localhost:8080
```

You should see the portfolio website.

At this point, Nginx is running inside Ubuntu WSL, and Windows can access the website through the browser.

---

## Part 5: Allow Other Students to SSH into the Server

Now other group members should connect to the server laptop using SSH.

### Step 1: Confirm SSH is Running in Ubuntu WSL

On the server laptop, inside Ubuntu WSL, run:

```bash
sudo service ssh status
```

If SSH is not running, start it:

```bash
sudo service ssh start
```

Confirm the SSH port being used:

```bash
sudo grep "^Port" /etc/ssh/sshd_config
```

If your previous lab used port `2222`, then students should connect using port `2222`.

### Step 2: Find the Windows Wi-Fi IP Address

On the server laptop, open PowerShell and run:

```powershell
ipconfig
```

Look for the Wi-Fi adapter.

Find the line called:

```text
IPv4 Address
```

It may look like:

```text
192.168.1.45
```

This is the IP address that other students will use to SSH into the server.

### Step 3: Find the WSL IP Address

Inside Ubuntu WSL, run:

```bash
hostname -I
```

You will see something like:

```text
172.25.45.120
```

Copy the first IP address.

This is the WSL IP address.

### Step 4: Forward SSH Traffic from Windows to WSL

Open PowerShell as Administrator on the server laptop.

If your SSH server in WSL uses port `2222`, run:

```powershell
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=2222 connectaddress=<WSL_IP> connectport=2222
```

Example:

```powershell
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=2222 connectaddress=172.25.45.120 connectport=2222
```

Allow SSH through Windows Firewall:

```powershell
New-NetFirewallRule -DisplayName "Allow WSL SSH 2222" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 2222
```

### Step 5: Other Students SSH into the Server

From another student’s Windows laptop, open PowerShell.

Run:

```powershell
ssh ubuntu_username@WINDOWS_WIFI_IP -p 2222
```

Example:

```powershell
ssh yves@192.168.1.45 -p 2222
```

Replace:

- `ubuntu_username` with the Ubuntu username on the server laptop.
- `WINDOWS_WIFI_IP` with the server laptop’s Wi-Fi IP address.

If the connection works, the student should now be inside the Ubuntu WSL environment of the server laptop.

They can test this by running:

```bash
pwd
whoami
hostname
```

---

## Part 6: Allow Other Students to Access the Website

Now the goal is for other students on the same school Wi-Fi to access the portfolio website in their browsers.

### Step 1: Forward Website Traffic from Windows to WSL

Open PowerShell as Administrator on the server laptop.

Use the same WSL IP address from the previous part.

Run:

```powershell
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=8080 connectaddress=<WSL_IP> connectport=8080
```

Example:

```powershell
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=8080 connectaddress=172.25.45.120 connectport=8080
```

Allow port `8080` through Windows Firewall:

```powershell
New-NetFirewallRule -DisplayName "Allow Nginx WSL 8080" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 8080
```

### Step 2: Classmates Open the Website

Other students should connect to the same school Wi-Fi.

Then they should open a browser and visit:

```text
http://WINDOWS_WIFI_IP:8080
```

Example:

```text
http://192.168.1.45:8080
```

They should now see the portfolio website hosted from one student’s laptop using Ubuntu WSL and Nginx.

### Step 3: Group Task

Once the website is working, each group should customize the portfolio.

Each group should update the website with:

- A team name.
- Names of all group members.
- A short “About Our Team” section.
- At least three project cards.
- Updated contact information.
- Better styling where needed.

Bonus:

- Add images.
- Add a separate page.
- Change colors and fonts.
- Add a group photo or team avatar.
