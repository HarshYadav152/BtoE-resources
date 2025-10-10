## Part 1: Installing Docker and Answer.dev

### Step 1: Install Docker
For Windows:
1. Download Docker Desktop from [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Run the installer and follow the prompts
3. Restart your computer when prompted

For Linux (Ubuntu/Debian):
```bash
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
# Log out and log back in for group changes to take effect
```

### Step 2: Create Answer.dev Directory and Configuration
```bash
# Create a directory for Answer
mkdir -p ~/answer-community
cd ~/answer-community

# Create docker-compose.yml file
cat > docker-compose.yml << 'EOF'
version: '3'
services:
  answer:
    image: answerdev/answer:latest
    restart: always
    ports:
      - '9080:80'
    volumes:
      - ./data:/data
EOF

# Start Answer.dev
docker-compose up -d
```

## Part 2: Port Forwarding Setup

### Step 3: Find Your Local IP Address
For Windows:
1. Open Command Prompt
2. Type `ipconfig` and press Enter
3. Note your IPv4 Address (something like 192.168.1.x)

For Linux:
```bash
ip addr show | grep "inet " | grep -v 127.0.0.1
```

### Step 4: Configure Port Forwarding on Your Router
1. Find your router's IP address (typically 192.168.0.1 or 192.168.1.1)
2. Open a web browser and enter this IP address
3. Log in with admin credentials (check router documentation if unknown)
4. Navigate to "Port Forwarding" or "Virtual Server" section
5. Create a new port forwarding rule:
   - External/Public Port: 80 (for HTTP)
   - Internal/Private Port: 9080
   - Internal IP Address: [Your computer's local IP from Step 3]
   - Protocol: TCP or Both (TCP+UDP)
6. Create another rule for HTTPS:
   - External/Public Port: 443
   - Internal/Private Port: 9080
   - Internal IP Address: [Same as above]
   - Protocol: TCP or Both (TCP+UDP)
7. Save and apply changes

### Step 5: Configure DNS for Your Domain
1. Log in to your domain registrar's website
2. Navigate to DNS settings for btoe.in
3. Create an A record:
   - Host: community
   - Value: [Your public IP address] (find it by googling "what is my ip")
   - TTL: Auto or 3600
4. Save changes (may take up to 24 hours to propagate)

## Part 3: Auto-Start Setup

### Step 6: Create Auto-Start Script
For Windows:
1. Create a batch file called `start-answer.bat`:

```batch
@echo off
cd %USERPROFILE%\answer-community
docker-compose up -d
```

2. Save this file in your answer-community folder

For Linux:
```bash
cat > ~/answer-community/start-answer.sh << 'EOF'
#!/bin/bash
cd ~/answer-community
docker-compose up -d
EOF

chmod +x ~/answer-community/start-answer.sh
```

### Step 7: Configure Auto-Start on Boot

For Windows:
1. Press Win+R, type `taskschd.msc` and press Enter
2. In Task Scheduler, click "Create Basic Task..."
3. Name it "Start Answer.dev"
4. Select "When the computer starts" as trigger
5. Choose "Start a program" as action
6. Browse to your `start-answer.bat` file
7. Finish the wizard
8. Right-click the newly created task, select "Properties"
9. Check "Run with highest privileges"
10. Click OK

For Linux:
```bash
crontab -e
```

Add this line:
```
@reboot ~/answer-community/start-answer.sh
```

Save and exit the editor

## Part 4: Verify Everything Works

### Step 8: Test Your Setup
1. Restart your computer
2. Wait a few minutes for Docker to start and containers to initialize
3. Open a web browser and go to http://localhost:9080 to verify Answer.dev is running locally
4. Visit http://community.btoe.in from another device (like your phone) to test external access

### Step 9: Complete Answer.dev Setup
1. Complete the initial setup wizard in Answer.dev
2. Create admin account
3. Configure your community settings
4. Customize branding

## Part 5: Security and Maintenance

### Step 10: Set Up Regular Updates
For Windows, create another Task Scheduler task to run weekly:
```batch
@echo off
cd %USERPROFILE%\answer-community
docker-compose pull
docker-compose down
docker-compose up -d
```

For Linux, create a cron job:
```bash
# Create update script
cat > ~/answer-community/update-answer.sh << 'EOF'
#!/bin/bash
cd ~/answer-community
docker-compose pull
docker-compose down
docker-compose up -d
EOF

chmod +x ~/answer-community/update-answer.sh

# Add to crontab to run every Sunday at 3am
(crontab -l; echo "0 3 * * 0 ~/answer-community/update-answer.sh") | crontab -
```
