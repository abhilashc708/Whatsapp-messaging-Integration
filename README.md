# Whatsapp-messaging-Integration
Whatsapp messaging integration on application

For integrating third party based whatsapp messaging integration on any web based application. The detailed steps are added below

**step 1**  :  Phase 1 – Install Node.js
Download the latest LTS version of Node.js from the official website and install it.
After installation, verify:
check node installed or not **node-v**, if yes we can see the version number.
               if not please install node first, detailed installation steps below

**step 2**  : Phase 2 – Create WhatsApp Service

Create a new folder outside your Spring Boot project.
eg: D:\TempleBilling
    backend
    frontend
    whatsapp-service

Open Command Prompt.

cd D:\TempleBilling\whatsapp-service

Initialize the project:

npm init -y

You will get   package.json

**Phase 3** – Install Packages

Install the required libraries:

npm install express

npm install whatsapp-web.js

npm install qrcode-terminal

npm install multer

npm install cors

npm install body-parser

npm install axios

Your package.json will include these dependencies.


**Phase 4** – Folder Structure

Inside whatsapp-service, create:

whatsapp-service
    node_modules
    uploads
    server.js
    package.json

The uploads folder will hold the PDFs sent from Spring Boot.

**Step 5** - Create server.js

Inside your whatsapp-service folder, create a file named:

server.js

Paste the following code.



const express = require("express");
const cors = require("cors");
const bodyParser = require("body-parser");
const qrcode = require("qrcode-terminal");

const { Client, LocalAuth } = require("whatsapp-web.js");

const app = express();

app.use(cors());
app.use(bodyParser.json());

const client = new Client({
    authStrategy: new LocalAuth({
        clientId: "temple-billing"
    }),
    puppeteer: {
        headless: true,
        args: [
            "--no-sandbox",
            "--disable-setuid-sandbox"
        ]
    }
});

client.on("qr", qr => {
    console.log("=================================");
    console.log("Scan this QR using Temple WhatsApp");
    console.log("=================================");
    qrcode.generate(qr, { small: true });
});

client.on("authenticated", () => {
    console.log("WhatsApp Authenticated");
});

client.on("ready", () => {
    console.log("=================================");
    console.log("WhatsApp Ready");
    console.log("=================================");
});

client.on("disconnected", reason => {
    console.log("Disconnected : " + reason);
});

client.initialize();

app.get("/status", (req, res) => {

    if (client.info) {

        return res.json({
            connected: true,
            number: client.info.wid.user
        });

    }

    res.json({
        connected: false
    });

});

app.listen(3000, () => {

    console.log("=================================");
    console.log("WhatsApp Server Started");
    console.log("http://localhost:3000");
    console.log("=================================");

});



**Step 6** - Run the server

Open Command Prompt.


cd D:\TempleBilling\whatsapp-service

Run   node server.js

You should see

WhatsApp Server Started

and then a QR code like

▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
█ ▄▄▄▄▄ ██ ▄▄▄▄▄ █
█ █   █ ██ █   █ █
...

**Step 7** - Scan QR

Open the Temple WhatsApp on the phone.


**Step 8** - Verify

After scanning you should see

WhatsApp Authenticated

WhatsApp Ready


**Step 9** - Test

Open your browser.

http://localhost:3000/status

{
   "connected": true,
   "number": "919876543210"
}


    
