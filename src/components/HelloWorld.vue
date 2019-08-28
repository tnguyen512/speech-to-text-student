<template lang="pug">
  <v-container>
    <v-layout>
        <label class="text-reader">
          <input type="file" @change="handleFile"> Import
        </label>
        <label class="text-reader">
            <input type="button" v-on:click="exportFile"> Export
        </label>
    </v-layout>
    <v-layout>
      <v-flex>
        <v-text-field  label="Tìm kiếm" solo v-model='search' v-on:click="toggleMarker"></v-text-field>
      </v-flex>
    </v-layout>
     <v-data-table :pagination.sync="pagination" :headers="headers" :items="filteredDatas" class="elevation-1">
    <template v-slot:items="probs" ref='index'>
      <td class="text-xs-left">{{ probs.index + 1 }}</td>
      <td class="text-xs-left">{{ probs.item.mssv }}</td>
      <td class="text-xs-left">{{ probs.item.name }}</td>
      <td class="text-xs-left">
      <input v-bind:id="'index'+ probs.index" v-on:focus="focused" type="text" label="Điểm" v-model="probs.item.score"/>
      </td>
    </template>
  </v-data-table>
  </v-container>
</template>

<script>
import XLSX from "xlsx";

export default {
  name: "HelloWorld",
  data() {
    return {
      pagination: {
        descending: false,
        page: 5,
        rowsPerPage: -1
      },
      search: "",
      temp: "",
      headers: [
        { text: "STT", value: "stt" },
        {
          text: "Mã số SV",
          value: "mssv"
        },
        { text: "Tên", value: "name" },
        { text: "Điểm", value: "score" }
      ],
      students: []
    };
  },
  computed: {
    filteredDatas() {
      var self = this;
      return this.students.filter(function(cust) {
        return cust.name.toLowerCase().indexOf(self.search.toLowerCase()) >= 0;
      });
    }
  },
  methods: {
    handleFile(e) {
      const files = e.target.files,
        f = files[0];
      const reader = new FileReader();
      const self = this;
      reader.onload = function(e) {
        var data = new Uint8Array(e.target.result);
        var workbook = XLSX.read(data, { type: "array" });
        const sheetJson = XLSX.utils.sheet_to_json(
          workbook.Sheets[workbook.SheetNames[0]],
          { raw: true, defval: null }
        );
        self.students = sheetJson;
        // self.filteredDatas = sheetJson;
      };
      reader.readAsArrayBuffer(f);
    },
    exportFile() {
      // export json to Worksheet of Excel
      // only array possible
      const studentSheet = XLSX.utils.json_to_sheet(this.students);

      // A workbook is the name given to an Excel file
      const wb = XLSX.utils.book_new(); // make Workbook of Excel

      // add Worksheet to Workbook
      // Workbook contains one or more worksheets
      XLSX.utils.book_append_sheet(wb, studentSheet, "sheet1"); // sheetAName is name of Worksheet

      // export Excel file
      XLSX.writeFile(wb, "students.xlsx"); // name of the file is 'book.xlsx'
    },
    focused() {
      window.SpeechRecognition =
        window.webkitSpeechRecognition || window.SpeechRecognition;
      const synth = window.speechSynthesis;
      const recognition = new SpeechRecognition();
      recognition.lang = "vi-VN";
      recognition.start();
      recognition.onresult = event => {
        const speechToText = event.results[0][0].transcript;
        console.log("set score " + speechToText);
        this.filteredDatas[0].score = this.convertTextToNumber(speechToText);
      };
    },
    toggleMarker() {
      console.log("speech");
      window.SpeechRecognition =
        window.webkitSpeechRecognition || window.SpeechRecognition;
      const synth = window.speechSynthesis;
      const recognition = new SpeechRecognition();
      recognition.lang = "vi-VN";

      this.search = "";

      recognition.start();
      recognition.onresult = event => {
        const speechToText = event.results[0][0].transcript;
        console.log(speechToText);
        this.search = speechToText;
        if (event.results[0].isFinal) {
          const studentSize = this.filteredDatas.length;
          if (studentSize == 1) {
            recognition.stop();
            document.getElementById("index0").focus();
          }
          if (studentSize > 1) {
            const utterThis = new SpeechSynthesisUtterance(
              "Tên sinh viên bị trùng"
            );
            utterThis.lang = "vi-VN";
            synth.speak(utterThis);
          }
        }
      };
    },
    convertTextToNumber(text) {
      switch (text) {
        case "một":
          return 1;
        case "hai":
          return 2;
        case "ba":
          return 3;
        case "bốn":
          return 4;
        case "năm":
          return 5;
        case "sáu":
          return 6;
        case "bảy":
          return 7;
        case "tám":
          return 8;
        case "chín":
        case "chính":
          return 9;
        case "mười":
          return 10;
        default:
          return text;
      }
    },
    getTime() {
      const time = new Date(Date.now());
      return `the time is ${time.toLocaleString("en-US", {
        hour: "numeric",
        minute: "numeric",
        hour12: true
      })}`;
    },

    getDate() {
      const time = new Date(Date.now());
      return `today is ${time.toLocaleDateString()}`;
    }
  }
};
</script>

<!-- Add "scoped" at'tribute to limit CSS to this component only -->
<style scoped>
#drop {
  border: 2px dashed #bbb;
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
  padding: 25px;
  text-align: center;
  font: 20pt bold, "Vollkorn";
  color: #bbb;
}
.text-reader {
  position: relative;
  overflow: hidden;
  display: inline-block;

  /* Fancy button style 😎 */
  border: 2px solid black;
  border-radius: 5px;
  padding: 8px 12px;
  cursor: pointer;
}
.text-reader input {
  position: absolute;
  top: 0;
  left: 0;
  z-index: -1;
  opacity: 0;
}
</style>
