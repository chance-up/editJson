<template>
  <div class="fixeded">
    <br />
    <b-row>
      <b-col cols="4">
        <b-button class="bv-example-btn" variant="warning" @click="uploadClick()">파일선택 </b-button>
        <input type="file" name="" class="file-input" accept=".json" @change="selectFile" ref="fileRef" hidden />
      </b-col>
      <b-col cols="4" v-if="selected !== ''">
        <b-button class="bv-example-btn" variant="warning" @click="save()">전체 JSON <br />업데이트/복사 </b-button>
      </b-col>
      <b-col cols="4" v-if="isSelectedFile">
        <b-button class="bv-example-btn" variant="warning" @click="search()">검색 </b-button>
      </b-col>
    </b-row>
    <br />
    <b-row v-if="selected !== ''">
      <b-col cols="4"><b-button class="bv-example-btn" variant="info" @click="back()">뒤로</b-button></b-col>
      <b-col cols="4"><b-button class="bv-example-btn" variant="primary" @click="addEmpty()">추가</b-button></b-col>
    </b-row>
    <br />
  </div>
  <div ref="scrollArea" class="scroll-area">
    <b-row v-if="isSelectedFile">
      <b-col cols="1.6" v-for="(category, idx) in categories" :key="idx">
        <b-button variant="outline-primary" @click="selectCategory(category)">{{ category }}</b-button>
      </b-col>
    </b-row>
    <br />

    <b-row v-if="selected !== ''">
      <b-col>
        <b-row v-for="(item, idx) in items" :key="idx">
          <b-col cols="5"><b-form-input id="input-small" size="sm" v-model="item[0]"></b-form-input></b-col>
          <b-col cols="5"><b-form-input id="input-small" size="sm" v-model="item[1]"></b-form-input></b-col>
          <b-col cols="1"><b-button variant="danger" @click="del(idx)">-</b-button></b-col>
          <b-col cols="1"><b-button variant="success" @click="lineCopy(idx)">copy</b-button></b-col>
          <p v-if="notiMessageDupl[idx].isValid === false" class="red-txt noti">{{ notiMessageDupl[idx].message }}</p>
        </b-row>
      </b-col>
    </b-row>
  </div>

  <SearchModal size="s" v-if="isShowSerchModal" style="z-index: 199">
    <template v-slot:modalHeader>
      <button @click="closeSearchModal">
        <i><img src="@/assets/close.svg" /></i>
      </button>
    </template>

    <template v-slot:modalContainer>
      <input type="text" id="dupl-id" class="input-box" :value="searchText" @input="changeSearchText" />
      <ul>
        <li v-for="(item, idx) in searchItems" :key="idx">
          <span class="badge" @click="copySearch(item[0], item[1])">{{
            item[0] + '.' + item[1] + ' : ' + item[3]
          }}</span>
        </li>
      </ul>
      <div class="pop-footer">
        <button class="lg-btn navy-btn" @click="closeSearchModal">닫기</button>
      </div>
    </template>
  </SearchModal>
</template>

<script setup lang="ts">
import { ref, watch, type Ref } from 'vue';
import useClipboard from 'vue-clipboard3';
import SearchModal from '@/components/SearchModal.vue';
import { useToast } from 'vue-toastification';

const { toClipboard } = useClipboard();
const toast = useToast();

type ValidationCheckType = {
  isValid: boolean | null;
  message: string;
};

const notiMessageDupl: Ref<ValidationCheckType[]> = ref([]);
const isDupl = ref(false);
const categories: Ref<any> = ref([]);
const isSelectedFile = ref(false);
const selected: Ref<string> = ref('');
const fullItems: Ref<any> = ref();
const items: Ref<any> = ref([]);
const isShowSerchModal = ref(false);
const searchText = ref('');
const searchItems: Ref<any> = ref([]);

watch(
  items,
  () => {
    initNotiMessageDupl();
    isDuplicate();
  },
  { deep: true }
);

watch(searchText, (v) => {
  searchItems.value = [];
  Object.entries(fullItems.value).forEach((el: any) => {
    Object.entries(fullItems.value[el[0]] as { [key: string]: any }).forEach((e) => {
      if (v !== '' && e[1].includes(v)) {
        searchItems.value.push([el[0], e[0], el[1], e[1]]);
      }
    });
  });
  console.log(searchItems.value);
});

const fileRef = ref();
const uploadClick = () => {
  fileRef.value.click();
};

const copySearch = async (s1: string, s2: string) => {
  try {
    let t = "{{ $t('" + s1 + '.' + s2 + "') }}";
    await toClipboard(t);
  } catch (e) {
    console.error(e);
  }
  closeSearchModal();
};

const changeSearchText = (e: Event) => {
  const target = e.target as HTMLSelectElement;
  searchText.value = target.value;
};

const search = () => {
  searchItems.value = [];
  isShowSerchModal.value = true;
};

const closeSearchModal = () => {
  searchText.value = '';
  searchItems.value = [];
  isShowSerchModal.value = false;
};

const selectFile = (event: Event) => {
  if (event.target != null) {
    const files = (event.target as HTMLInputElement).files;
    if (files != null && files.length > 0) {
      if (!files[0].name.includes('.json')) {
        alert('잘못된 File 형식입니다.');
        return;
      }
      let reader = new FileReader();
      reader.readAsText(files[0] as Blob);
      reader.onload = (e) => {
        const source = reader.result as string;
        fullItems.value = JSON.parse(source);
        categories.value = Object.keys(fullItems.value);
        isSelectedFile.value = true;

        //파일 이벤트 초기화
        const fileElement = document.getElementsByClassName(
          (event.target as HTMLInputElement).className
        )[0] as HTMLInputElement;
        fileElement.value = '';
      };
    }
  }
};

const selectCategory = (category: string) => {
  selected.value = category;
  items.value = jsonToArr(fullItems.value, category);
  items.value.forEach((item: any) => {
    notiMessageDupl.value.push({
      isValid: null,
      message: '',
    });
  });
};

const lineCopy = async (idx: number) => {
  try {
    let t = "{{ $t('" + selected.value + '.' + items.value[idx][0] + "') }}";
    await toClipboard(t);
  } catch (e) {
    console.error(e);
  }
};

function isDuplicate() {
  let duplIdx = 0;
  for (let i = 0; i < items.value.length; i++) {
    for (let j = i + 1; j < items.value.length - 1; j++) {
      if (items.value[i][0] == items.value[j][0]) {
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

const back = () => {
  selected.value = '';
};

const save = async () => {
  if (isDupl.value) {
    toast.warning('중복된 값이 있습니다.');
    return;
  }

  let partObj = arrToJson(items.value);
  fullItems.value[selected.value as keyof typeof fullItems.value] = partObj;
  try {
    await toClipboard(JSON.stringify(fullItems.value));
  } catch (e) {
    alert(e);
  }
};
const jsonToArr = (json: any, category: string) => {
  return Object.entries(json[category] as { [key: string]: any }).map((el) => {
    return [el[0], el[1]];
  });
};

const arrToJson = (arr: any) => {
  console.log(arr);
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
  padding: 10px;
  border-radius: 10px;
  outline: auto;
  font-size: 16px;
  width: 40%;
  height: 100%;
}
.file-input {
  padding: 1rem;
  font-size: 16px;
  width: 40%;
  height: 100%;
}

.fixeded {
  background: rgb(40, 0, 148);
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
.badge {
  display: inline-block;
  padding: 0.35em 0.65em;
  font-size: 12px;
  font-weight: 700;
  line-height: 1;
  color: #fff;
  text-align: center;
  white-space: nowrap;
  vertical-align: baseline;
  border-radius: 0.25rem;
  background-color: #ece1fe;
  color: #8041f7 !important;
}
</style>
