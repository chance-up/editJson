<template>
  <div class="fixeded">
    <b-row v-if="selected !== ''">
      <b-col cols="3"><b-button class="bv-example-btn" variant="info" @click="back()">뒤로</b-button></b-col>
      <b-col cols="3"><b-button class="bv-example-btn" variant="primary" @click="addEmpty()">추가</b-button></b-col>
      <b-col cols="3"><b-button class="bv-example-btn" variant="success" @click="copy()">저장</b-button></b-col>
    </b-row>
  </div>
  <div ref="scrollArea" class="scroll-area">
    <b-row v-if="selected === ''">
      <b-col cols="2" v-for="(category, idx) in categories" :key="idx">
        <b-button variant="outline-primary" @click="selectCategory(category)">{{ category }}</b-button>
      </b-col>
    </b-row>
    <b-row v-if="selected !== ''">
      <b-col
        ><b-row v-for="(item, idx) in items" :key="idx">
          <b-col cols="5"><b-form-input id="input-small" size="sm" v-model="item[0]"></b-form-input></b-col>
          <b-col cols="5"><b-form-input id="input-small" size="sm" v-model="item[1]"></b-form-input></b-col>
          <b-col cols="1"><b-button variant="danger" @click="del(idx)">-</b-button></b-col>
          <p v-if="notiMessageDupl[idx].isValid === false" class="red-txt noti">{{ notiMessageDupl[idx].message }}</p>
        </b-row>
      </b-col>
    </b-row>
  </div>
</template>

<script setup lang="ts">
import json from '../../vue3-admin/src/locales/kr.json';
import { onMounted, ref, watch, type Ref } from 'vue';
import useClipboard from 'vue-clipboard3';
type ValidationCheckType = {
  isValid: boolean | null;
  message: string;
};
const notiMessageDupl: Ref<ValidationCheckType[]> = ref([]);
const isDupl = ref(false);
const categories: Ref<any> = ref([]);
const selected: Ref<string> = ref('');
const items: Ref<any> = ref([]);
onMounted(() => {
  categories.value = Object.keys(json);
});

watch(
  items,
  () => {
    initNotiMessageDupl();
    isDuplicate();
  },
  { deep: true }
);
function isDuplicate() {
  let duplIdx = 0;
  for (let i = 0; i < items.value.length; i++) {
    for (let j = i + 1; j < items.value.length - 1; j++) {
      if (items.value[i][0] == items.value[j][0]) {
        console.log();
        duplIdx = j;
        notiMessageDupl.value[j].isValid = false;
        notiMessageDupl.value[j].message = '중복입니다~';
        isDupl.value = true;
        break;
      }
    }
  }
  return duplIdx;
}
const initNotiMessageDupl = () => {
  isDupl.value = false;
  for (let i = 0; i < notiMessageDupl.value.length; i++) {
    notiMessageDupl.value[i].isValid = true;
    notiMessageDupl.value[i].message = '';
  }
};

const selectCategory = (category: string) => {
  selected.value = category;
  items.value = jsonToArr(json, category);
  items.value.forEach((item: any) => {
    notiMessageDupl.value.push({
      isValid: null,
      message: '',
    });
  });
};
const back = () => {
  selected.value = '';
};
const { toClipboard } = useClipboard();
const copy = async () => {
  if (isDupl.value) {
    alert('중복된 값이 있습니다~');
    return;
  }
  try {
    let partObj = arrToJson(items.value);
    json[selected.value as keyof typeof json] = partObj;
    await toClipboard(JSON.stringify(json));
    console.log('Copied to clipboard');
  } catch (e) {
    console.error(e);
  }
};
const jsonToArr = (json: any, category: string) => {
  return Object.entries(json[category] as { [key: string]: any }).map((el) => {
    return [el[0], el[1]];
  });
};

const arrToJson = (arr: any) => {
  let str = '';
  str += '{';
  arr.forEach((e: any) => {
    if (e[0] !== '' && e[1] !== '') {
      str += '"' + e[0] + '":"' + e[1] + '"';
      str += ',';
    }
  });
  str = str.slice(0, -1);
  str += '}';
  return JSON.parse(str);
  // return str;
};
const scrollArea = ref();
const addEmpty = async () => {
  notiMessageDupl.value.push({
    isValid: null,
    message: '',
  });
  items.value.push(['', '']);
  scrollArea.value.scrollIntoView({ block: 'end' });
};
const del = (idx: number) => {
  items.value.splice(idx, 1);
  console.log(items.value[idx]);
};
</script>

<style scoped>
header {
  line-height: 1.5;
}

.bv-example-row {
  margin: 2rem;
  padding: 3rem;
}

.bv-example-btn {
  padding: 1rem;
  font-size: 16px;
  width: 40%;
  height: 100%;
}

.fixeded {
  background: deeppink;
  color: white;
  padding-left: 32px;
  padding-right: 32px;
  justify-content: space-between;
  align-items: center;
  position: sticky;
  top: 0;
  z-index: 100;
}
.scroll-area {
  padding-top: 2rem;
  padding-bottom: 2rem;
}

.red-txt {
  display: inline-block;
  position: relative;
  font-size: 14px;
  font-weight: 400;
  line-height: 24px;
  color: #ff4e63;
  text-align: left;
  margin: 0px 0px;
}

div.con {
  width: 1000px;
  margin: 100px auto;
  margin-bottom: 1000px;
  height: 10000px;
  background: linear-gradient(to bottom, red, yellow);
}
</style>
