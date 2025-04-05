<script setup>
import HelloWorld from './components/HelloWorld.vue'
import { Printer } from 'capacitor-plugin-serialprinter';
import { LEDControl } from 'capacitor-plugin-ledcontrol';

Printer.connectPrinter()

async function print2() {
  await Printer.connectPrinter()
  Printer.setAlignment({ align: 2 });
  Printer.setFontSize({ size: 4 });
  Printer.printText({ data: `the POS test` });
  Printer.setAlignment({ align: 1 });
  for (let i = 0; i < 10; i++) {
    Printer.setFontSize({ size: i });
    Printer.printText({ data: `TESTING ${i}` });
  }
  Printer.setFontSize({ size: 2 });
  Printer.setAlignment({ align: 2 });
  Printer.printText({ data: "********************************" })
  Printer.setAlignment({ align: 1 });
  Printer.printQRCode({ data: 'https://printpos.invaliduser.xyz', size: 400 });
  Printer.printText({ data: "table" });
  //TODO: implement table function


  Printer.cutPaper()
  Printer.printBarcode({ data: "202403250032" })
}

async function printTable(tableData) {
  const paperWidth = 80; // Width of the paper in mm
  const totalColumns = tableData[0].length;
  const columnSpacing = 1; // Space between columns
  const maxColumnWidth = Math.floor(paperWidth / totalColumns) - columnSpacing;

  let columnWidths = [];

  // Determine the maximum width of each column, capped at maxColumnWidth
  tableData.forEach(row => {
    row.forEach((cell, index) => {
      if (!columnWidths[index]) columnWidths[index] = 0;
      columnWidths[index] = Math.min(Math.max(columnWidths[index], cell.length), maxColumnWidth);
    });
  });

  // Print each row with proper alignment
  for (const rowData of tableData) {
    let formattedRow = '';
    rowData.forEach((cell, index) => {
      const padding = ' '.repeat(columnWidths[index] - cell.length + columnSpacing);
      formattedRow += `${cell}${padding}`;
    });
    await Printer.printText({ data: formattedRow });
    await Printer.newLine(); // Move to the next line
  }
}



Printer.addListener("printerStatusChanged", (stat) => {
  console.log("status chuang");
  console.log(stat);
})

</script>


<script>
import { CapacitorHttp } from '@capacitor/core';
import { Printer } from 'capacitor-plugin-serialprinter';
import { LEDControl } from 'capacitor-plugin-ledcontrol';

export default {
  data() {
    return {
      imgUrl: "https://fastly.picsum.photos/id/1047/536/354.jpg?hmac=Hqs-Rz08WiLc2elw4gHvY1P-wxDJfmiZ-CSay2BH-1U",
      printerConnected: false,
      led: {
        orange: false,
        blue: false
      },
      textBold: false,
      textPosition: 1,
      textSize: 1,
      textToPrint: "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Phasellus euismod gravida nisi, et rhoncus urna iaculis sit amet. Proin rhoncus ullamcorper semper. Pellentesque habitant morbi tristique senectus et netus et malesuada fames ac turpis egestas. In eget magna dui. Cras et porta purus. Nulla accumsan ac felis sit amet tempus. Curabitur arcu lorem, suscipit nec elementum eu, aliquet ut sem. Ut sollicitudin tincidunt molestie. Duis a tempor risus, a finibus enim. Ut tempor elementum purus, vel aliquam dolor volutpat id. Nulla nec rhoncus elit. Aenean in consectetur urna. Mauris at orci suscipit, sodales augue vitae, posuere felis. Donec sed semper massa.",
      qrCodeSize: 200,
      QRCodeValue: "https://postest.invaliduser.xyz",
      tableData: [
        ["beef lasagna", "£3.00", "1", "£3.00"],
        ["sandwich", "£3.50", "2", "£7.00"],
        ["sml hodogs, beans chips dinner for 1", "£7.59", "1", "£7.59"],
        ["DR. pepper", "£1.20", "3", "£3.60"],
        ["fresh air", "£50", "999", "£49950"],
      ],
      barcodeData: "910000201370"
    }
  },
  beforeCreate() {
    LEDControl.turnOffBlueLED();
    LEDControl.turnOffRedLED();
    this.led.orange = false;
    this.led.blue = false;
    Printer.setFontSize({ size: this.textSize });
    Printer.setAlignment({ align: 1 });
  },
  methods: {
    setTextPos(val) {
      this.textPosition = val;
      Printer.setAlignment({ align: this.textPosition });
      return;
    },
    setTextSize(val) {
      this.textSize = val;
      Printer.setFontSize({ size: this.textSize });
      return;
    },
    async toggleLEDOrange() {
      if (this.led.orange == true) {
        LEDControl.turnOffRedLED();
        this.led.orange = false;
      } else {
        LEDControl.turnOnRedLED();
        this.led.orange = true;
      }
    },
    async toggleLEDBlue() {
      if (this.led.blue == true) {
        LEDControl.turnOffBlueLED();
        this.led.blue = false;
      } else {
        LEDControl.turnOnBlueLED();
        this.led.blue = true;
      }
    },
    connectPrinter() {
      Printer.connectPrinter();
    },
    textPrint() {
      Printer.printUTF8({ data: this.textToPrint });
      return;
    },
    cutPaper() {
      Printer.cutPaper();

    },
    setQRSize(val) {
      this.qrCodeSize = val;
    },
    QRPrint() {
      Printer.printQRCode({ data: this.QRCodeValue, size: this.qrCodeSize });
      return;
    },
    barCodePrint() {
      Printer.printBarcode({ data: this.barcodeData });
      return;
    },
    async imagePrint(url) {
      try {

        const canvas = document.getElementById('imgcanvas');
        const ctx = canvas.getContext('2d');
        const img = new Image();

        img.crossOrigin = 'Anonymous';
        img.onload = () => {
          canvas.width = img.width;
          canvas.height = img.height;
          ctx.drawImage(img, 0, 0);

          // Convert the canvas image to a base64 string
          const base64Image = canvas.toDataURL();
          console.log(base64Image)

          Printer.printImage({ data: base64Image.split(',')[1] });
        }
        img.src = url;
      } catch (error) {
        console.error('Error fetching image:', error);
        return null; // Return null if there's an error
      }

    },
    createSimplePriceTable(itemsArr, printCharLength = 48) {

      function makeBar() {
        let bar = '';
        for (let i = 0; i < printCharLength; i++) {
          bar += '-';
        }
        return bar;
      }
      let outputStr = '             ITEM           PRICE   QTY   AMOUNT' + makeBar();
      for (let row of itemsArr) {
        if (row.length > 1 && row.length < 4) {
          throw new Error(row.toString() + " must not contain any less than 2 or any more than 4 strings");
        }
        console.log('\n\n\n' + row);
        const item = row[0];
        const price = row[1];
        const qty = row[2];
        const amount = row[3];
        let startOfRowStr = "";
        let endOfRowStr = `  ${price}   ${qty}   ${amount}`;
        let charsLeft = printCharLength - endOfRowStr.length;

        if (item.length <= charsLeft) {
          let lengthLeft = charsLeft - item.length;
          startOfRowStr = item;
          if (lengthLeft > 0) {
            for (let i = 0; i < lengthLeft; i++) {
              startOfRowStr += ' ';
            }
          }
        } else if (item.length > charsLeft) {
          startOfRowStr = item.slice(0, charsLeft - 3) + '...';
        }
        outputStr += `${startOfRowStr}${endOfRowStr}`;
      }
      outputStr += makeBar();
      return outputStr;
    },
    printSimplePriceTable() {
      this.setTextPos(1);
      this.setTextSize(1);
      const prices = this.createSimplePriceTable(this.tableData);
      Printer.printUTF8({ data: prices });
    },
    addNewTableRow() {
      this.tableData.push(['item', 'price', 'qty', 'amount']);
    },
    removeRow(index) {
      this.tableData.splice(index, 1);
    },
    openCashDraw() {
      Printer.openCashDrawer();
    }
  }
}
</script>


<template>

  <div class="paper"></div>
  <h1>H10-1 POS test</h1>

  <h2>LED Bar</h2>
  <button @click="toggleLEDBlue()" style="background-color: blue;">Toggle Blue LED</button>
  <button @click="toggleLEDOrange()" style="background-color: red;">Toggle red/orange LED</button>

  <h2>Printer</h2>

  <p>Status: <span>{{ printerConnected == true ? 'online' : 'offline' }}</span></p>
  <button @click="connectPrinter()">Connect Printer</button>
  <button @click="cutPaper()">Cut Paper</button>

  <h3>Text Print</h3>
  <textarea v-model="textToPrint"></textarea>
  <button @click="textPrint()">Print UTF-8 text</button>

  <p>Position:</p>
  <div>
    <button @click="setTextPos(1)" :style="[textPosition == 1 ? 'color:grey' : '']">left</button>
    <button @click="setTextPos(2)" :style="[textPosition == 2 ? 'color:grey' : '']">center</button>
    <button @click="setTextPos(3)" :style="[textPosition == 3 ? 'color:grey' : '']">center</button>
  </div>

  <p>Size:</p>
  <div>
    <button @click="setTextSize(1)" :style="[textSize == 1 ? 'color:grey' : '']">XSmall (default)</button>
    <button @click="setTextSize(2)" :style="[textSize == 2 ? 'color:grey' : '']">Small</button>
    <button @click="setTextSize(3)" :style="[textSize == 3 ? 'color:grey' : '']">medium</button>
    <button @click="setTextSize(4)" :style="[textSize == 4 ? 'color:grey' : '']">Large (title)</button>
  </div>

  <h3>Barcode Print</h3>
  <input type="text" v-model="barcodeData" />
  <button @click="barCodePrint()">Print Barcode</button>

  <h3>QR Print</h3>
  <input type="text" v-model="QRCodeValue" />
  <button @click="QRPrint()">Print QR code</button>
  <p>Set Size:</p>
  <div>
    <button @click="setQRSize(200)" :style="[qrCodeSize == 200 ? 'color:grey' : '']">200</button>
    <button @click="setQRSize(400)" :style="[qrCodeSize == 400 ? 'color:grey' : '']">400</button>
  </div>

  <h3>Photo Print</h3>
  <canvas id="imgcanvas"></canvas>
  <input v-model="imgUrl" />
  <button @click="imagePrint(imgUrl)">Print IMG</button>

  <h3>Table Print</h3>
  <p>NOTE: table print is not a native function but a custom one currently the only one i have is based on the demo one
    in the pos util app</p>

  <h3>Simple Price Table</h3>
  <table>
    <thead>
      <tr>
        <th>Item</th>
        <th>Price</th>
        <th>Quantity</th>
        <th>Amount</th>
        <th></th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(row, index) in tableData" :key="index">
        <td><input v-model="row[0]" type="text"></td>
        <td><input v-model="row[1]" type="text"></td>
        <td><input v-model="row[2]" type="text"></td>
        <td><input v-model="row[3]" type="text"></td>
        <td><button @click="removeRow(index)">Remove</button></td>
      </tr>
    </tbody>
  </table>
  <button @click="addNewTableRow()">Add Row</button>

  <button @click="printSimplePriceTable()">Print simple price table</button>
  <h3>cash draw</h3>
  <button @click="openCashDraw()">Open</button>


</template>

<style scoped>
.paper {
  background-color: #FFFFFF;
  border: 0.9px solid black;
  width: 410px;
  margin: auto 0;

  min-height: 100px;
}

.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
