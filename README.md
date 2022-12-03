                Bot-whatsapp

Installation
* npm i express moment exceljs
* npm i whatsapp-web.js qrcode-terminal
* Node V16+ es requerido


For example:

```js
const fs = require('fs');
const exceljs = require('exceljs');
const moment = require('moment');

const { Client, MessageMedia } = require('whatsapp-web.js');
const qrcode = require('qrcode-terminal');

let client;

client = new Client();

client.on('qr', (qr) => {
    qrcode.generate(qr, { small: true });
});

client.on('ready', () => {
    console.log('Cliente is ready');
    listenMessage();
});

const listenMessage = () => {
    client.on('message', async msg => {
        const { from, body } = msg;
        if (msg.hasMedia) {
            const media = await msg.downloadMedia();
            console.log(media);
        } else {
            const contact = await msg.getContact();
            switch (body) {
                case 'hola' || 'Hola': sendMessage(from, `Hola @${contact.id.user}! Éste es mi bot de whatsapp`);
                    sendMessage(from, '¿En que puedo ayudarte?');
                    sendMedia(from, 'sticker-meme.webp'); break;
                case 'chau' || 'Chau': sendMessage(from, '¡Espero que mi bot haya sido de ayuda! Nos vemos :)'); break;
                default: console.log('el mensaje no se reconoce')
            }
        }
        saveHistorial(from, body);
    });
}

const sendMessage = (to, message) => {
    client.sendMessage(to, message);
}

const sendMedia = (to, file) => {
    const mediaFile = MessageMedia.fromFilePath(`./media/${file}`);
    client.sendMessage(to, mediaFile);
}

const saveHistorial = (number, message) => {
    const pathChat = `./chats/${number}.xlsx`;
    const workbook = new exceljs.Workbook();
    const today = moment().format('DD-MM-YYYY hh:mm');

if (fs.existsSync(pathChat)) {
        workbook.xlsx.readFile(pathChat)
            .then(() => {
                const worksheet = workbook.getWorksheet(1);
                const lastRow = worksheet.lastRow;
                let getRowInsert = worksheet.getRow(++(lastRow.number));
                getRowInsert.getCell('A').value = today;
                getRowInsert.getCell('B').value = message;
                getRowInsert.commit();
                workbook.xlsx.writeFile(pathChat)
                    .then(() => {
                        console.log('Se agrego el chat');
                    })
                    .catch(() => {
                        console.log('No se pudo agregar al chat');
                    })
            })
    } else {
        const worksheet = workbook.addWorksheet('Chats');
        worksheet.columns = [
            { header: 'Fecha', key: 'date' },
            { header: 'Mensaje', key: 'message' },
        ]
        worksheet.addRow([today, message]);
        workbook.xlsx.writeFile(pathChat)
            .then(() => {
                console.log('Historial Creado!')
            })
            .catch(() => {
                console.log('No se pudo crear el historial!')
            })
    }
}

client.initialize();
```
