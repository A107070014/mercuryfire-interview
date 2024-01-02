<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input ref="nameRef" v-model="tempData.name" label="姓名 *"
          :rules="[val => (val && val.length > 0) || '請輸入姓名']" />
        <q-input ref="ageRef" type="number" v-model="tempData.age" label="年齡 *" :rules="[
          val => (val !== null && val !== '') || '請輸入年齡',
          val => (val >= 0) || '年齡必須大於零'
        ]" />
        <q-btn color="primary" class="q-mt-md" @click="validateForm">{{ isEdit ? '更新' : '新增' }}</q-btn>
      </div>

      <q-table flat bordered ref="tableRef" :rows="blockData" :columns="(tableConfig as QTableProps['columns'])"
        row-key="id" hide-pagination separator="cell" :rows-per-page-options="[0]">
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td v-for="col in props.cols" :key="col.name" :props="props" style="min-width: 120px">
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn @click="handleClickOption(btn, props.row)" v-for="(btn, index) in tableButtons" :key="index"
                size="sm" color="grey-6" round dense :icon="btn.icon" class="q-ml-md" padding="5px 5px">
                <q-tooltip transition-show="scale" transition-hide="scale" anchor="top middle" self="bottom middle"
                  :offset="[10, 10]">
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div class="full-width row flex-center items-center text-primary q-gutter-sm" style="font-size: 18px">
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps, useQuasar } from 'quasar';
import { ref, onMounted } from 'vue';
const $q = useQuasar();
interface btnType {
  label: string;
  icon: string;
  status: string;
}
interface dataType {
  id: string,
  creatorId: string,
  name: string,
  age: number | string,
}
interface tableConfigType {
  label: string,
  name: string,
  field: string,
  align: string,
}
const blockData = ref<dataType[]>([
  {
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref<tableConfigType>([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref<btnType[]>([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tableData = ref<dataType[]>([]);
onMounted(() => {
  getAllData();
})
async function getAllData() {
  try {
    const resData = await axios.get("/crudTest/a");
    tableData.value = resData.data;
  } catch (error) {
    console.log("fetch error:", error);
  }
}

const isEdit = ref<boolean>(false);
const tempData = ref<dataType>({
  name: '',
  age: '',
});
const nameRef = ref(null);
const ageRef = ref(null);

function handleClickOption(btn: btnType, data: dataType) {
  console.log("handleClickOption", btn, data);
  switch (btn.status) {
    case 'edit':
      isEdit.value = true;
      tempData.value.name = data.name;
      tempData.value.age = data.age;
      break;

    case 'delete':
      handleDeleteData(data.id);
      break;
  }
  // ...
}
function handleDeleteData(id: string) {
  $q.dialog({
    title: '提示',
    message: '是否確定刪除該筆資料?',
    cancel: true,
    persistent: true
  }).onOk(async () => {
    // console.log('>>>> OK')
    try {
      const resData = await axios.delete(`/crudTest/${id}`);
      getAllData();
    } catch (error) {
      console.log("fetch error:", error);
    }
  }).onOk(async () => {
    // console.log('>>>> second OK catcher')
    try {
      const resData = await axios.delete(`/crudTest/${id}`);
      getAllData();
    } catch (error) {
      console.log("fetch error:", error);
    }
  }).onCancel(() => {
    // console.log('>>>> Cancel')
  }).onDismiss(() => {
    // console.log('I am triggered on both OK and Cancel')
  })
}

function validateForm() {
  nameRef.value.validate();
  ageRef.value.validate();

  if (nameRef.value.hasError || ageRef.value.hasError) return

  if (isEdit.value) {
    updateData();
  } else {
    createData();
  }
}
async function createData() {
  console.log("createData", tempData);
  try {
    const resData = await axios.post("/crudTest", tempData.value);
    tempData.value.name = "";
    tempData.value.age = "";
    getAllData();
  } catch (error) {
    console.log("fetch error:", error);
  }
}
async function updateData() {
  console.log("updateData", tempData);
  try {
    const resData = await axios.patch("/crudTest", tempData.value);
    tempData.value.name = "";
    tempData.value.age = "";
    isEdit.value = false;
    getAllData();
  } catch (error) {
    console.log("fetch error:", error);
  }
}
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
