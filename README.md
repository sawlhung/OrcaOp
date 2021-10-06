# OrcaOp
@@ -1,14 +1,52 @@
# Overview
This is a project completed for a challege for FileMaker internship. API data manipulation written in Node.js and express with modern syntax Promise, acync, await. Tic Tac Toe game built in FileMaker 17. 

# Buit With
## Buit With
- [42SV data api](https://api.intra.42.fr/apidoc) - Fetch student info data
- [FileMaker 17 data api](https://fmhelp.filemaker.com/docs/17/en/dataapi/index.html) - post to Filemaker server
- [nexmo api](https://dashboard.nexmo.com/getting-started-guide) -  send message to game loser


#Diagram
### Diagram
![diagram](res/apiDiagram.png)


## Installation
### Download Filemaker and FileMaker server
Please install `FileMaker Pro 17 Advanced` and `FileMaker Server 17` on-premises. 
Host the [TicTacToe game](https://github.com/Lijun21/FileMaker-42-TicTacToe/blob/master/TicTacToeGame.fmp12) to your server.

### Clone source code
```shell
git clone https://github.com/Lijun21/FileMaker-42-TicTacToe.git
```

### Install Js Dependencies
```
npm install
```

### Configuration
create a `config.js` file under the same folder, and copy paste code below.
```
module.exports = {
    clientID42 : 'YOUR_42_CLIENT_ID',
    clientSecret42 : 'YOUR_42_CLIENT_SECRET',
    fmHostURI: 'YOUR_FileMaker_SERVER_URI',
    fmFileName: "Lijun_TicTacToe",
    fmLoginPass: 'bGlqdW46MTIzNA==',
    fmLayoutName: 'Player',
    nexmoKey: 'YOUR_NEXMO_KEY',
    nexmoSecret: 'YOUR_NEXMO_SECRET',
    testModeNumber: '12013713874'
}
```

### Run the server
```
npm start
```




  1  package.json 
@@ -12,7 +12,6 @@
  },
  "devDependencies": {},
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js"
  },
  "repository": {
  15  routes/fileMaker.js 
@@ -1,19 +1,20 @@
var request = require('request');
const keys = require('../config');

process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";

const host_URI = 'https://10.114.1.8';
const databaseName = 'Lijun_TicTacToe';
const login_pw = 'bGlqdW46MTIzNA=='; //login:password base 64 format
const layoutName = 'Player';
const fmHostURI = keys.fmHostURI;
const fmFileName = keys.fmFileName;
const fmLoginPass = keys.fmLoginPass; //login:password base 64 format
const fmLayoutName = keys.fmLayoutName;

//POST request, to get access token from FM data API
function getToken(){
    var optionsGetToken = {
        url: `${host_URI}/fmi/data/v1/databases/${databaseName}/sessions`,
        url: `${fmHostURI}/fmi/data/v1/databases/${fmFileName}/sessions`,
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Basic ${login_pw}`
            'Authorization': `Basic ${fmLoginPass}`
        }
    };
    return new Promise((resolve, reject) => {
@@ -32,7 +33,7 @@ function getToken(){
//POST request, send student_info to FileMake via FM data API
module.exports = async function PostDataToFM(student_info) {
    const token = await getToken();
    var url = `${host_URI}/fmi/data/v1/databases/${databaseName}/layouts/${layoutName}/records/`;
    var url = `${fmHostURI}/fmi/data/v1/databases/${fmFileName}/layouts/${fmLayoutName}/records/`;
    var optionsPostData = {
      method: 'post',
      body: student_info,
  2  routes/school42.js 
@@ -81,7 +81,7 @@ module.exports = (app) => {
    const messageContent = 'yo, I beat you up in Tic Tac Toe game!!! Hahahahahah.....';
    app.get('/message', (req, res) => {
        nexmo.message.sendSms(
            keys.myPhoneNumber, `1${student_info.phone}`, messageContent,
            keys.testModeNumber, `1${student_info.phone}`, messageContent,
                (err, responseData) => {
                if (err) {
                    console.log(err);
