<script setup lang="ts">
import {computed, onMounted, ref} from 'vue';
import Popup from '~/components/AdminInfoPopup.vue'
import {useNuxtApp} from "#app";

let {$axios} = useNuxtApp()

interface User {
  id: number;
  username: string;
  password: string;
  name: string;
  position: string;
  staff_ID: string;
  phone: string;
  email: string;
  staff_type: string | null;
}

const columns = [
  { key: 'id', label: 'ID' },
  { key: 'username', label: 'Username' },
  { key: 'name', label: 'Name' },
  { key: 'position', label: 'Position' },
  { key: 'staff_ID', label: 'Staff ID' },
  { key: 'phone', label: 'Phone' },
  { key: 'staff_type', label: 'Staff Type' },
  { key: 'extend', label: 'View', sortable: false },
];

const admins = ref<User[]>([]);
const currentPage = ref(1);
const pageSize = ref(10);
const totalItems = ref(0);
const q = ref('');

const api = $axios()

const fetchData = async () => {
  try {
    const response = await api.get("/users/");
    admins.value = response.data.map((admin: any, index: number) => ({
      id: index + 1,
      username: admin.username || '',
      password: admin.password || '',
      name: admin.profile?.name || '',
      position: admin.profile?.position || '',
      staff_ID: admin.profile?.staff_ID || '',
      phone: admin.profile?.phone || '',
      email: admin.profile?.email || '',
      staff_type: admin.profile?.staff_type || null,
    }));
    totalItems.value = response.data.length;
  } catch (error) {
    console.error('Error fetching data:', error);
  }
};

const isPopupVisible = ref(false);
const currentStudent = ref({});

onMounted(fetchData)

definePageMeta({
  middleware: 'auth',
});

const openPopup = (row: User) => {
  currentStudent.value = row;
  isPopupVisible.value = true;
};

const filteredRows = computed(() => {
  if (!q.value) {
    return admins.value;
  }

  return admins.value.filter((admin) => {
    return Object.values(admin).some((value) => {
      return String(value).toLowerCase().includes(q.value.toLowerCase());
    });
  });
});

const paginatedRows = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  return filteredRows.value.slice(start, end);
});

const handlePageChange = (newPage: number) => {
  currentPage.value = newPage;
};

onMounted(fetchData)

</script>

<template>
  <div class="admin-dashboard">
    <div class="container">
      <main class="dashboard-content">
        <div class="sub-container">

          <div class="content">
            <div class="header">

              <div class="search-container">
                <UInput v-model="q" placeholder="Filter admins..."/>
              </div>
            </div>

            <UTable :columns="columns" :rows="paginatedRows">
              <template #extend-data="{ row }">
                <a @click="openPopup(row)" class="extend-btn">View</a>
                <Popup
                    :show="isPopupVisible"
                    @update:show="isPopupVisible = $event"
                    :admins="currentStudent"
                />
              </template>
            </UTable>
            <div class="pagination">
              <button
                  :disabled="currentPage === 1"
                  @click="handlePageChange(currentPage - 1)"
              >
                <UIcon
                    name="mdi-arrow-left"
                />
              </button>
              <span>Page {{ currentPage }} of {{ Math.ceil(totalItems / pageSize) }}</span>
              <button
                  :disabled="currentPage >= Math.ceil(totalItems / pageSize)"
                  @click="handlePageChange(currentPage + 1)"
              >
                <UIcon
                    name="mdi-arrow-right"
                />
              </button>
            </div>


            <hr class="divider"/>
            <div class="footer">
              <h2 class="footer-megs" style="text-align: center">Thank you !</h2>
            </div>
          </div>
        </div>
      </main>

    </div>
  </div>
</template>

<style scoped>
.admin-dashboard {
  display: block;
  background-color: var(--main-color);
}

.container {
  display: flex;
  flex-wrap: nowrap;
  padding: 0;
  border-top: 3px solid var(--text-hovor-color);
  border-bottom: 3px solid var(--text-hovor-color);
  width: 100%;
  margin: 0 auto;
}

.dashboard-content {
  flex: 6;
}

@media (max-width: 1200px) {
  .admin-dashboard {
    display: block;
  }

}

.sidebar ul li {
  list-style: none;
  margin: 0.5rem;
  padding: 0.5rem;
  font-size: 1rem;
  text-align: start;
  text-transform: capitalize;
  font-weight: normal;
  color: var(--text-hovor-color);
  background-color: transparent;
}

.sidebar li:hover {
  color: var(--text-hovor-color);
  background-color: var(--main-hovor-color);
  transition: .3s ease-in-out;
}

.dashboard-content {
  flex: 10;
  background-color: #eeeeee;
}

.dashboard-info-content div {
  margin: 1rem;
}

.dashboard-info-content div {
  margin: 1rem;
}

.dashboard-content .header {
  display: inline-flex;
  flex-wrap: wrap;
  margin: 0.5rem;
  align-items: center;
}

.extend-btn {
  padding: .5rem;
  border-radius: .5rem 0;
  color: var(--text-hovor-color);
  background-color: var(--main-hovor-color);
  cursor: pointer;
}

.extend-btn:hover {
  color: var(--text-color);
  background-color: var(--main-color);
  transition: .3s ease-in-out;
}

.header h2,
.footer h2 {
  font-size: 1.5rem;
  color: var(--main-hovor-color);
  text-align: center;
  margin: 1rem auto;
}

.divider {
  border-bottom: 2px solid var(--main-hovor-color);
  margin: 1rem 0;
}

.pagination {
  display: flex;
  justify-content: center;
  margin: 1rem 0;
}

.pagination span {
  padding: .5rem 1rem;
  border-radius: .5rem;
  transition: 0.3s ease-in-out;
}

.pagination button {
  padding: .5rem;
  border-radius: .5rem;
  color: var(--text-color);
  background-color: var(--main-color);
  transition: 0.3s ease-in-out;
}

@media (max-width: 1200px) {
  .container {
    display: block;
  }
}

@media (max-width: 768px) {
  .dashboard-content {
    padding: 1rem;
  }
}


</style>
