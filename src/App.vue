<script setup>
import { ref } from "vue";
import csvToJson from "convert-csv-to-json";

const lineTotal = 11;
let csvJson = [];
let jaUser = [];
let avallonUser = [];
let khyterraUser = [];

const pollOptionName = ref({
  ja: "JA",
  avallon: "Avallon",
  khyterra: "Khyterra",
});
const msg = ref([]);
const spiderRingFormation = ref("");
const simpleText = ref(false);

const splitPollOption = () => {
  if (!csvJson.length) return;
  csvJson.forEach((csv) => {
    let user = simpleText.value ? csv.username : `${csv.name} ${csv.username}`;
    if (csv.answer === pollOptionName.value.khyterra) {
      khyterraUser.push(user);
    } else if (csv.answer === pollOptionName.value.avallon) {
      avallonUser.push(user);
    } else {
      jaUser.push(user);
    }
  });
};

const getSpiderRingFormation = () => {
  const statusMsg = [];
  if (khyterraUser.length < lineTotal) {
    statusMsg.push("Kekurangan Personil Khyterra");
  }
  if (avallonUser.length < lineTotal) {
    statusMsg.push("Kekurangan Personil Avallon");
  }
  if (jaUser.length < lineTotal) {
    statusMsg.push("Kekurangan Personil JA");
  }
  if (statusMsg.length) {
    msg.value = statusMsg;
  }
  const innerRingTotal = Math.floor(jaUser.length / lineTotal);
  let ir = 0;
  const spiderRingArray = [];
  for (let i = 0; i < lineTotal; i++) {
    const lineUser = [
      khyterraUser[i]
        ? `${khyterraUser[i]}${simpleText.value ? "" : " (Khyterra)"}`
        : `${simpleText.value ? "" : "(Khyterra)"}`,
    ];
    if (innerRingTotal === 0) {
      lineUser.push(jaUser[ir] ? jaUser[ir] : "");
      ir++;
    } else {
      for (let j = 0; j < innerRingTotal; j++) {
        lineUser.push(jaUser[ir] ? jaUser[ir] : "");
        ir++;
      }
    }
    lineUser.push(
      avallonUser[i]
        ? `${avallonUser[i]}${simpleText.value ? "" : " (Avallon)"}`
        : `${simpleText.value ? "" : "(Avallon)"}`
    );
    spiderRingArray.push(lineUser);
  }
  let formationStr = "";
  for (let i = 0; i < spiderRingArray.length; i++) {
    if (i > 0) {
      formationStr += "\n";
    }
    if (!simpleText.value) {
      formationStr += `Line ${i + 1}\nRing 1 (Tim 12)\n`;
    }
    for (let j = 0; j < spiderRingArray[i].length; j++) {
      const ringText = simpleText.value ? "R" : "Ring ";
      formationStr += `${ringText}${j + 2} ${spiderRingArray[i][j]}\n`;
    }
  }
  const restOfJaUser = innerRingTotal > 0 ? jaUser.length % lineTotal : 0;
  if (
    khyterraUser.length > lineTotal ||
    avallonUser.length > lineTotal ||
    restOfJaUser !== 0
  ) {
    formationStr += "\nPengamat\n";
    if (khyterraUser.length > lineTotal) {
      formationStr += "\nKhyterra:\n";
      let n = 1;
      for (let i = lineTotal - 1; i < khyterraUser.length; i++) {
        formationStr += `${n}. ${khyterraUser[i]}\n`;
        n++;
      }
    }
    if (avallonUser.length > lineTotal) {
      formationStr += "\nAvallon:\n";
      let n = 1;
      for (let i = lineTotal - 1; i < avallonUser.length; i++) {
        formationStr += `${n}. ${avallonUser[i]}\n`;
        n++;
      }
    }
    if (restOfJaUser !== 0) {
      formationStr += "\nJA:\n";
      let n = 1;
      for (let i = lineTotal * innerRingTotal - 1; i < jaUser.length; i++) {
        formationStr += `${n}. ${jaUser[i]}\n`;
        n++;
      }
    }
  }
  spiderRingFormation.value = formationStr;
};

const onFileChange = (event) => {
  const file = event.target.files[0];
  const reader = new FileReader();
  reader.readAsText(file);
  reader.onload = (res) => {
    const csvStrJson = csvToJson
      .supportQuotedField(true)
      .fieldDelimiter(",")
      .csvStringToJson(res.target.result);
    csvJson = csvStrJson
      .map((csv) => {
        const name = [csv['"first_name"'], csv['"last_name"']].join(" ");
        const timestamp = new Date(csv['"timestamp"']).getTime();
        return {
          answer: csv['"answer"'],
          username: `@${csv['"username"']}`,
          name,
          timestamp,
        };
      })
      .sort((a, b) => {
        return a.timestamp - b.timestamp;
      });
  };
};

const onGenerate = () => {
  if (!csvJson.length) {
    msg.value = ["file csv belum dipilih"];
    return;
  }
  if (
    !pollOptionName.value.khyterra ||
    !pollOptionName.value.avallon ||
    !pollOptionName.value.ja
  ) {
    msg.value = ["nama pilihan poll wajib diisi !!!"];
    return;
  }
  msg.value = [];
  jaUser = [];
  avallonUser = [];
  khyterraUser = [];
  splitPollOption();
  getSpiderRingFormation();
};
</script>

<template>
  <div>
    <h1>Generator Spider Ring</h1>
    <h4>
      Made by <a href="https://t.me/wildan_yr" target="_blank">WildanYR</a>
    </h4>
    <h4>
      Thanks to
      <a href="https://t.me/jurnalastral" target="_blank">Jurnal Astral</a>,
      <a href="https://t.me/MemperhatikanRasa" target="_blank"
        >Memperhatikan Rasa~</a
      >, and
      <a href="https://t.me/PollStatsBot" target="_blank">Poll Stats Bot</a>
    </h4>
  </div>
  <div>
    <p style="margin-bottom: 2px; color: #424242; font-size: 0.9rem">
      Pilih File .csv dari Poll Stats Bot
    </p>
    <input type="file" accept=".csv" @change="onFileChange" />
  </div>
  <div>
    <p style="margin-bottom: 2px; color: #424242; font-size: 0.9rem">
      Nama Pilihan/opsi JA di Polling
    </p>
    <input
      v-model="pollOptionName.ja"
      type="text"
      placeholder="nama pilihan poll JA"
    />
  </div>
  <div>
    <p style="margin-bottom: 2px; color: #424242; font-size: 0.9rem">
      Nama Pilihan/opsi Avallon di Polling
    </p>
    <input
      v-model="pollOptionName.avallon"
      type="text"
      placeholder="nama pilihan poll Avallon"
    />
  </div>
  <div>
    <p style="margin-bottom: 2px; color: #424242; font-size: 0.9rem">
      Nama Pilihan/opsi Khyterra di Polling
    </p>
    <input
      v-model="pollOptionName.khyterra"
      type="text"
      placeholder="nama pilihan poll Khyterra"
    />
  </div>
  <button
    @click="onGenerate"
    style="margin-top: 1rem; background-color: #292929; color: white"
  >
    Generate
  </button>
  <div>
    <div style="display: flex; gap: 0.25rem; align-items: center">
      <input type="checkbox" v-model="simpleText" />
      <p>Teks formasi simpel</p>
    </div>
  </div>
  <template v-if="msg.length">
    <h3>INFO</h3>
    <p v-for="(m, i) in msg" :key="'info-' + i">{{ m }}</p>
  </template>
  <p v-else></p>
  <div>
    <textarea
      v-model="spiderRingFormation"
      cols="50"
      rows="20"
      readonly
    ></textarea>
  </div>
</template>
