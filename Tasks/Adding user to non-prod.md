&nbsp;

# 🚀 Steps to Add a New User on `nprod` Server (10.4.56.233)

1.  **SSH into the server**
    
    ```bash
    ssh raushan_T4468@10.4.56.233 #this is our jumppost
    ssh ec2-user@ip-to-give-acess
    
    
    
    
    
    
    ```
    
2.  **Switch to root**
    
    ```bash
    sudo su -
    ```
    
3.  **Go to home directory (to check existing users)**
    
    ```bash
    cd /home
    ls -l
    ```
    
4.  **Create the new user (example: `user_T4468`)**
    
    ```bash
    useradd user_T4468
    ```
    
5.  **Create `.ssh` directory**
    
    ```bash
    cd /home/user_T4468
    mkdir .ssh
    chmod 700 .ssh
    ```
    
6.  **Add the authorized public key**
    
    ```bash
    cd .ssh
    vim authorized_keys
    ```
    
    > Paste the user’s **public key** inside this file.
    
7.  **Set correct permissions**
    
    ```bash
    chmod 600 authorized_keys
    ls -lrt
    ```
    
8.  **Set ownership of home directory**
    
    ```bash
    chown -R user_T4468:user_T4468 /home/user_T4468
    ```
    
9.  **(Optional: give sudo access)**  
    If the user needs sudo:
    
    ```bash
    usermod -aG wheel user_T4468
    ```
    

* * *

✅ Now `user_T4468` can log in using their private key:

```bash
ssh user_T4468@10.4.56.233
```

* * *

### View All Members of the Wheel Group

```
getent group wheel
```

&nbsp;

&nbsp;