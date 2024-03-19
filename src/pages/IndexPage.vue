<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="formData.name" label="姓名" :rules="[val => !!val || '必填欄位']" />
        <q-input v-model="formData.email" label="信箱" :rules="[val => validateEmail(val) || 'Email 不合法']"/>
        <q-btn color="primary" class="q-mt-md" @click="handleSubmit()">{{ !editMode ? '新增' : '更新' }}</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
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
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleSubmit(btn, props)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                :color="btn.color"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
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
import { QTableProps } from 'quasar';
import { ref } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref<Record<string, any>[]>([]);
const editMode = ref(false);
const editingUserId = ref(null);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '電子信箱',
    name: 'email',
    field: 'email',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
    color:'secondary'
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
    color:'deep-orange'
  },
]);

const formData = ref({name: 'name', email: 'demo@a.cc',});

const handleSubmit = (btn?: btnType, data?: Record<string,any>)=>{
  if(btn){
    const { status } = btn
    if(status === 'edit'){
      const { row } = data as Record<string, any>;
      editUser(row)
      return
    }

    if(status === 'delete'){
      const { row, rowIndex } = data as Record<string, any>;
      deleteUser(row.id, rowIndex)
      return
    }
  }else{
    editMode.value ? updateUser() : addUser()
  }
}

const resetForm = () => {
  formData.value.email = '';
  formData.value.name = '';
  editMode.value = false;
  editingUserId.value = null;
};

const apiDomain = 'https://jsonplaceholder.typicode.com';
const validateEmail = (email: string) => {
  const re = /\S+@\S+\.\S+/;
  return re.test(email);
};

const loadUsers = async () => {
  try{
      const { data } = await axios.get(`${apiDomain}/users?_limit=5`)
      blockData.value = data;
  }catch(e){
      console.log(e)
  }
};

const addUser = async() => {
    try{
        if (!validateEmail(formData.value.email) || !formData.value.name.trim() ) {
          return;
        }

        const { data } = await axios.post(`${apiDomain}/users`, {
            name: formData.value.name,
            email: formData.value.email
        })
        blockData.value.push({...data, id: Math.floor(Math.random()*10000)});
        
        resetForm();
    }catch(e){
        console.log(e)
    }
};

const editUser = (user: Record<string, any>) => {
  formData.value.email= user.email;
  formData.value.name = user.name;
  editingUserId.value = user.id;
  editMode.value = true;
};

const updateUser = async() => {
    try{
        if (!validateEmail(formData.value.email) || !formData.value.name.trim() ) {
            return;
        }

        if (editingUserId.value !== null) {
            const { data } = await axios.patch(`${apiDomain}/users/${editingUserId.value}`, {
                name: formData.value.name,
                email: formData.value.email
            })

            const index = blockData.value.findIndex(user => user.id === editingUserId.value);

            if(index > -1){
                blockData.value[index] = { ...blockData.value[index], ...data };
                resetForm()
            }
        }
    }catch(e){
        console.log(e)
    }
};

const deleteUser = async(id:string, index:number) => {
    try{
        await axios.delete(`${apiDomain}/users/${id}`)
        blockData.value.splice(index, 1);
        resetForm()
    }catch(e){
        console.log(e)
    }
};

loadUsers();
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
