[electron installation](#electron_installation)    
[Displaying a Window](#Displaying_a_Window)   
[Managing Windows](#Managing_Windows) 


<a name="electron_installation"/>  
# electron installation(for linux )(001)


1. frist you need install nodejs on your computer  
   `$ sudo dnf install nodejs`

  for check versions  
  `$ node --version`  
  `$ npm --version`

2. to new start Project  
    `$ npm init `  

3. install electron package to nodejs  
`$ npm install electron --save`    

4. then open package.json file and change "scripts" value

   ```json
   "scripts": {
     "start": "electron --disable-gpu app.js"
   }
   ```
  * we set script file that's why we need only `$ npm start` to run our Project  

 for disable GL ERROR and use CPU
 `--disable-gpu `
 
 
<a name="Displaying_a_Window"/>   
 # Displaying a Window(003)

to open window we just need to use following code  

```javascript
const electron = require('electron');
const {app,BrowserWindow} = electron;

app.on('ready',()=>{
  let win = new BrowserWindow({width:800,height:600});
  win.loadURL(`file://${__dirname}/index.html`);
});
```  
following line we need to set the location(in Acute) of html document.

```javascript
   win.loadURL(`file://${__dirname}/index.html`);
 ```
 
 <a name="Managing_Windows"/>   
 #  Managing Windows(004)

to show new window we have to create new html file and its need to connect with main(app.js) file


* version 5, the default for `nodeIntegration` changed from true to false.
we have to fix it true. otherwice its show `require is not defined`
```javascript
mainWindow = new BrowserWindow({
    webPreferences: {
        nodeIntegration: true
    }
});
```

* we have to careful when we close old window. first we
have to change current window to new window  and then only have to close old window.

 ```javascript
  var window = remote.getCurrentWindow(); // load current
  main.openWindow("page001"); // change current to new window
  window.close(); // close the "window" variable window

 ```
* this is how to make in main.js(app.js) function that can call from another js file
that include html file.
 we have to create like this function because all window functions are
 working on main.js(app.js) file.
  ```javascript
  exports.openWindow = (fileName)=>{
    let win = new BrowserWindow({width:800,height:600});
    win.loadURL(`file://${__dirname}/`+fileName+`.html`);
  };
  ```

* remote function that connect with main(app.js)
   ```javascript
   const remote = require('electron').remote;
   const main = remote.require('./app.js'); // get main

   var button = document.createElement('button');
   button.textContent = 'Open Window';
   button.addEventListener('click',()=>{
     var window = remote.getCurrentWindow();
     main.openWindow("page001"); // execute function on main(app.js)
     window.close();
   },false);
   document.body.appendChild(button);
  ```
  in this case `remote` is electron construtor(as I think)

# Packaging Applications(app) (005)

* this is like .jar file in Java.we can open this project
just clicking icon .

* todo this you need to download some packages to same directory.

 `$ npm install electron-packager --save-dev`
  this packages insert to node_module

* to safe app can user following package call "asar"
  `$  npm install asar --save-dev`


* then you need to change package.json file's script part
 following way.
 ```javascript
 "scripts": {
   "start": "electron --disable-gpu   app.js",
   "package":"",
   "build":"electron-packager . SabaragamuPawn"
}
 ```
 `SabaragamuPawn` is software(app) name.

 * to build the package
 `$ npm run build`

 * to delete build file completely
 `$ rm -rf SabaragamuPawn-linux-x64`

## this not work on fedora and tute have to do
 
 # Electron Reload(008)

* when we change html file and we have to rerun 'npm start' see result of changes.

* there are package call [electron-reload](https://www.npmjs.com/package/electron-reload) to do this job easily.It will show update code results(changes) only we save the changes.

* to install [electron-reload](https://www.npmjs.com/package/electron-reload) package  
`$ npm install electron-reload --save`

* to active this package you need to add following code to main(app.js)  

 ```javascript
  'use strict'; // add new

  const {app, BrowserWindow} = require('electron');

  require('electron-reload')(__dirname); // add new
  
  
  # print Files(009)

### print  PDF (electron: 7.1.9)

 * in new version of electron there are return "Promiss" `win.webContents.printToPDF()` of this
 function
  render.html
  ```javascript
  const ipc = require('electron').ipcRenderer; // get ipc lib
  const printPDFButton = document.getElementById('printDoc'); // get buttion element

  printPDFButton.addEventListener('click',function(event){
    ipc.send('print-to-pdf');
  },false);

  ipc.on('wrote-pdf',function(event,path){
    const message = `Wrote PDF to: ${path}`;
    console.log('message',message);
  });

  ```

  main (app.js)
  ```javascript
  ipc.on('print-to-pdf', (event, arg) => {
    //const pdfPath = `${app.getPath('desktop')}/${arg.name}.pdf`
    const pdfPath = path.join(os.tmpdir(),'print3.pdf');
    const win = BrowserWindow.fromWebContents(event.sender);
    win.webContents.printToPDF({
      pageSize: 'Letter' ,printBackground: true, landscape: false
    }).then(data => {
      //console.log("pdf data :",data);
      fs.writeFile(pdfPath,data,function(err){ // writeFile
        if(err) return console.log(err,message);
        shell.openExternal('file://'+pdfPath);
        event.sender.send('wrote-pdf', pdfPath);
      });
    }
    )
  });
  ```
 open created file following codes shows
 `shell.openExternal('file://'+pdfPath);`

 return data (files path) to display message. this is <mark>ipc</mark> communication in parallel programing.
  `event.sender.send('wrote-pdf', pdfPath);`

### print using printer (electron: 7.1.9)
* let's see how to print useing printer
  there no Promiss and you just have to print and you can remove above return <mark>ipc</mark>
  rander.html
  ```javascript
  const ipc = require('electron').ipcRenderer;
  const printPDFButton = document.getElementById('printDoc');

  printPDFButton.addEventListener('click',function(event){
    ipc.send('print-to-document');
  },false);
```

  main (app.js)
  ```javascript
   ipc.on('print-to-document', (event, arg) => {

   const win = BrowserWindow.fromWebContents(event.sender);
   win.webContents.print({
     pageSize: 'Letter' , printBackground: false, landscape: false
   });

  });
```
# Building an Installer on Windows(015)

* for this you need to install <mark>electron-builder</mark> as globally.  
`$ sudo npm install -g electron-builder`


* for quiek build instraller for Windows(fedora don't worked)  
 `$ build -w`

* you can build using package.json file.And for that you have to edit it.
```javascript
"scripts":{
  "start":"electron .",
  "pack" : "build --dir"
},
"build":{
  "appId":"com.pawn.tutorial.project",
  "productName":"Sabaragamu Pawn"
}
```
"scripts": {
 "start": "electron --disable-gpu   app.js",
 "pack" : "build --dir",
 "package": "",
 "build": "electron-packager . SabaragamuPawn "
},



* install <mark>electron-builder</mark> globally and locally. And run following command on project directory.(keep electron only dependencies not dependencies on package.json.otherwise <mark>electron-builder</mark> package will added to setup file too )

* to see <mark>electron-builder</mark>  commands  
 `$ electron-builder --help`

* make installer for linux
 `electron-builder build --linux`

* as a result  
