# Prestashop-Deployed-On-AWS-Free-Tier-With-Separate-RDS-Database


# PrestaShop on AWS Free Tier with Separate RDS Database

![PrestaShop](https://img.shields.io/badge/PrestaShop-8.1.6-31B8BB?style=flat-square&logo=prestashop)
![AWS](https://img.shields.io/badge/AWS-Free%20Tier-F90?style=flat-square&logo=amazonaws)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)

A **fully functional PrestaShop 8.1.6** e-commerce platform deployed on **AWS Free Tier** using:

- **EC2 t3.micro** (Ubuntu 22.04 LTS) – Web server (Apache + PHP 8.1)
  
- **RDS MySQL 8.0** (db.t3.micro) – **Separate database** (security & scalability)
  
- **Publicly accessible URL** via EC2 Public DNS
  
- **All resources within AWS Free Tier limits**


## Public Storefront URL

> `http://ec2-16-171-64-104.eu-north-1.compute.amazonaws.com`


## Architecture Overview

```
Internet
↓ (HTTP:80)
EC2 Instance (t3.micro)
├─ Apache 2 + PHP 8.1
├─ PrestaShop 8.1.6
└─ /var/www/html (files)
↓ (MySQL:3306 – private)
RDS Instance (db.t3.micro)
└─ MySQL 8.0 (prestashop_db)
```

- **Security Groups**:
- EC2: `HTTP (80)` from `0.0.0.0/0`, `SSH (22)` from your IP
- RDS: `MySQL (3306)` **only from EC2 security group**
- **No local database** — fully separated as required



## Step-by-Step Setup (Documented)

### 1. Launch EC2 Instance
- AMI: **Ubuntu Server 22.04 LTS**
- Instance: **t3.micro** (Free Tier)
- Key pair: `prestashop-key.pem`
- Security Group (`launch-wizard-X`):
- Inbound: `SSH (22)` → My IP
- Inbound: `HTTP (80)` → Anywhere
- Auto-assign public IP: **Enabled**

### 2. Install LAMP Stack
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 php8.1 libapache2-mod-php8.1 php8.1-mysql \
php8.1-curl php8.1-gd php8.1-intl php8.1-mbstring \
php8.1-xml php8.1-zip php8.1-bcmath unzip wget -y
sudo a2enmod rewrite
sudo systemctl restart apache2
```

### 3. Create RDS MySQL Instance
- Engine: **MySQL 8.0**
- Template: **Free Tier**
- Identifier: `prestashop-rds`
- Username: `ps_user`
- Database: `prestashop_db`
- Public access: **No**
- Connect to EC2 → Auto-configures security group

### 4. Deploy PrestaShop Files
```bash
cd /var/www/html
sudo wget https://github.com/PrestaShop/PrestaShop/releases/download/8.1.6/prestashop_8.1.6.zip
sudo unzip prestashop_8.1.6.zip
sudo mv prestashop/* .
sudo rmdir prestashop
sudo rm prestashop_8.1.6.zip
```

### 5. Set Permissions
```bash
sudo chown -R www-data:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
sudo mkdir -p var/cache var/logs img cache log upload download
sudo chmod -R 777 var img cache log upload download themes modules translations override config
```

### 6. Configure Apache
```bash
sudo nano /etc/apache2/mods-enabled/dir.conf
```
→ Set: `DirectoryIndex index.php ...`

```bash
sudo systemctl restart apache2
```

### 7. Run PrestaShop Installer
- URL: `http://your-ec2-dns/install`
- Database:
- Server: `prestashop-rds.xxxx.us-east-1.rds.amazonaws.com`
- Name: `prestashop_db`
- User: `ps_user`
- Password: (RDS master password)
- Test connection → **Connected**
- Set shop name, admin email, strong password
- **Admin URL generated** (e.g., `/admin123456`)

### 8. Finalize & Secure
```bash
sudo rm -rf /var/www/html/install
```

---

## Screenshots Included (in `screenshots/` folder)

| File | Description |
|------|-----------|
| `ec2-launch.png` | EC2 instance with public DNS |
| `rds-creation.png` | RDS with EC2 connection |
| `security-groups.png` | Inbound/outbound rules |
| `apache-dir-conf.png` | `DirectoryIndex index.php` |
| `installer-db.png` | RDS connection success |
| `terminal-permissions.png` | `chmod`, `chown`, `ls -la` |
| `final-storefront.png` | Live PrestaShop homepage |


## Security & Best Practices

- `install/` folder **deleted**
- Admin URL **randomized**
- MySQL **not exposed publicly**
- All ports restricted via security groups
- Using **Free Tier only**



## AWS Resources Used

| Service | Resource | Free Tier Eligible |
|--------|---------|---------------------|
| EC2 | t3.micro | Yes |
| RDS | db.t3.micro, 20 GB | Yes |
| EBS | 8 GB gp3 | Yes |

**Total cost: $0** (within Free Tier)





## Author

**John Ofulue/Neptune430**
*November 2025*


**PrestaShop is now live, secure, and ready for demo.**
```
