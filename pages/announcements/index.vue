<script setup>
import announcementApi from "@/api/announcementApi";
import busRoutesApi from "~/api/busRoutesApi";
const router = useRouter();
const route = useRoute();
const loadList = ref([1, 2, 3, 4, 5]);
const announcements = ref([]);
const loading = ref(false);
const totalPage = ref(0);
const tab = ref(2);
const currentSort = ref("appartment_status");
let transports = [];
const sortOptions = [
  {
    name: "Eng yangilari",
    id: 1,
    options: ["appartment_status", "appartment_status_reverse"],
  },
  {
    name: "Tavsiya etilganlar",
    id: 2,
    options: ["price_for_one", "price_for_one_reverse"],
  },
  {
    name: "Narx bo’yicha",
    id: 3,
    options: ["total_price", "total_price_reverse"],
  },
];

async function filterSend(e) {
  let query = { ...route.query };
  Object.entries(e).forEach(([name, value]) => {
    query[name] = value;
  });
  await router.replace({
    path: "/announcements",
    query: { ...query },
  });
  __GET_ANNOUNCEMENTS();
}
async function clearFilter() {
  await router.replace({
    path: "/announcements",
    query: {},
  });
  currentSort.value = "appartment_status";
  __GET_ANNOUNCEMENTS();
}
const errorHandle = (error) => {
  error.response
    ? ElNotification({
        title: "Error",
        message: error.response.statusText,
        type: "error",
      })
    : ElNotification({
        title: "Error",
        message: "Serverda xatolik",
        type: "error",
      });
};
async function __GET_ANNOUNCEMENTS() {
  try {
    loading.value = true;
    const data = await announcementApi.getAnnouncement({
      params: { ...route.query, page_size: route.query.page_size || 6 },
    });
    announcements.value = data?.data?.results;
    totalPage.value = data?.data?.count;
  } catch (e) {
    errorHandle(e);
  } finally {
    loading.value = false;
  }
}
async function __GET_TRANSPORT() {
  try {
    loading.value = true;
    const data = await busRoutesApi.getTransport();
    transports = data?.data;
  } catch (e) {
    console.log(e);
    errorHandle(e);
  } finally {
    loading.value = false;
  }
}
function sortHandle(option) {
  currentSort.value == option[0]
    ? (currentSort.value = option[1])
    : (currentSort.value = option[0]);
  filterSend({ ordering: currentSort.value });
}
useAsyncData("announcement", async () => {
  return __GET_ANNOUNCEMENTS();
});
onMounted(() => {
  if (route.query?.ordering) {
    currentSort.value = route.query?.ordering;
  }
  __GET_TRANSPORT();
});
</script>
<template>
  <div class="applications py-[60px]">
    <div class="2xl:container mx-auto px-4">
      <div class="flex items-center justify-between mb-[30px]">
        <h1 class="font-600 text-[60px] flex items-center gap-[24px]">
          Natija:
          <span class="text-[color:var(--green)] flex items-center">{{ totalPage }}</span>
        </h1>
        <button
          @click="$router.push('/map')"
          class="bg-[color:var(--btn-blue)] text-white py-[10px] flex rounded-[8px] min-w-[250px] justify-center"
        >
          Haritada ko’rish
          <span
            ><svg
              xmlns="http://www.w3.org/2000/svg"
              width="25"
              height="24"
              viewBox="0 0 25 24"
              fill="none"
            >
              <path
                fill-rule="evenodd"
                clip-rule="evenodd"
                d="M9 9C9 6.92893 10.6789 5.25 12.75 5.25C14.8211 5.25 16.5 6.92893 16.5 9C16.5 11.0711 14.8211 12.75 12.75 12.75C10.6789 12.75 9 11.0711 9 9ZM12.75 6.75C11.5074 6.75 10.5 7.75736 10.5 9C10.5 10.2426 11.5074 11.25 12.75 11.25C13.9926 11.25 15 10.2426 15 9C15 7.75736 13.9926 6.75 12.75 6.75Z"
                fill="white"
              />
              <path
                fill-rule="evenodd"
                clip-rule="evenodd"
                d="M6.20691 8.12724C6.48067 4.80603 9.25605 2.25 12.5885 2.25H12.9125C16.245 2.25 19.0203 4.80603 19.2941 8.12724C19.4408 9.90751 18.8909 11.6753 17.7602 13.0581L14.1654 17.4545C13.4341 18.3488 12.0669 18.3488 11.3356 17.4545L7.74082 13.0581C6.61008 11.6752 6.06017 9.90751 6.20691 8.12724ZM12.5885 3.75C10.0367 3.75 7.91147 5.70726 7.70184 8.25046C7.58702 9.64343 8.0173 11.0266 8.90204 12.1086L12.4968 16.505C12.6279 16.6653 12.8731 16.6653 13.0042 16.505L16.599 12.1086C17.4837 11.0266 17.914 9.64343 17.7992 8.25046C17.5895 5.70726 15.4643 3.75 12.9125 3.75H12.5885Z"
                fill="white"
              />
              <path
                d="M8.42082 16.3353C8.60606 15.9648 8.45589 15.5143 8.08541 15.3291C7.71493 15.1438 7.26442 15.294 7.07918 15.6645L5.07918 19.6645C4.96293 19.897 4.97536 20.1731 5.11201 20.3942C5.24867 20.6153 5.49007 20.7499 5.75 20.7499H19.75C20.0099 20.7499 20.2513 20.6153 20.388 20.3942C20.5246 20.1731 20.5371 19.897 20.4208 19.6645L18.4208 15.6645C18.2356 15.294 17.7851 15.1438 17.4146 15.3291C17.0441 15.5143 16.8939 15.9648 17.0792 16.3353L18.5365 19.2499H6.96353L8.42082 16.3353Z"
                fill="white"
              /></svg
          ></span>
        </button>
      </div>
      <div class="applications-container grid">
        <div class="flex flex-col gap-6">
          <AppSearch @getData="__GET_ANNOUNCEMENTS" :transports="transports" />
          <AppFilter
            @filter="filterSend"
            @clear-filter="clearFilter"
            :loading="loading"
          />
        </div>
        <div>
          <div class="flex items-center justify-between mb-[30px]">
            <h6 class="font-600 text-[color:#C5C6C7] text-[20px]">Tahlash:</h6>
            <div
              class="tab gap-[10px] flex px-[4px] rounded-[12px] items-center border border-[color:#EAEAEA] border-solid py-[4px]"
            >
              <button
                v-for="sort in sortOptions"
                :key="sort.id"
                class="px-[20px] whitespace-nowrap py-[10px] rounded-[12px] text-[color:var(--dark-1)] text-[16px] font-500 flex items-center gap-[10px]"
                :class="{ active: sort.options.includes(currentSort) }"
                @click="sortHandle(sort.options)"
              >
                {{ sort.name }}
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="18"
                  height="18"
                  viewBox="0 0 18 18"
                  fill="none"
                >
                  <g clip-path="url(#clip0_347_11118)">
                    <path
                      v-if="sort.options[0] == currentSort"
                      d="M8.96211 5.9625L7.90161 7.023L5.99961 5.121V15H4.49961V5.121L2.59836 7.023L1.53711 5.9625L5.24961 2.25L8.96211 5.9625Z"
                      fill="#50BFE8"
                    />
                    <path
                      v-if="sort.options[0] == currentSort"
                      d="M8.96211 5.9625L7.90161 7.023L5.99961 5.121V15H4.49961V5.121L2.59836 7.023L1.53711 5.9625L5.24961 2.25L8.96211 5.9625Z"
                      fill="white"
                    />
                    <path
                      v-else
                      d="M12.7496 15.75L16.4621 12.0375L15.4016 10.977L13.4996 12.879V3H11.9996L12.0004 12.879L10.0976 10.977L9.03711 12.0375L12.7496 15.75Z"
                      fill="white"
                    />
                  </g>
                  <defs>
                    <clipPath id="clip0_347_11118">
                      <rect width="18" height="18" fill="white" />
                    </clipPath>
                  </defs>
                </svg>
              </button>
            </div>
          </div>

          <div class="flex flex-col gap-[31px]" v-if="loading">
            <el-skeleton
              v-for="loadCard in loadList"
              :key="loadCard"
              :rows="0"
              width="100%"
            />
          </div>
          <div class="flex flex-col gap-[31px]" v-else>
            <AppTheCard
              v-for="announcement in announcements"
              :key="announcement.id"
              :announcement="announcement"
            />
          </div>
          <div class="flex justify-center mt-6">
            <VPagination
              :load="true"
              class="xl:hidden"
              :totalPage="totalPage"
              @getData="__GET_ANNOUNCEMENTS"
            />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="css" scoped>
:deep(.el-skeleton__p.is-first) {
  width: 100%;
  border-radius: 18px;
  height: 311px;
}
.applications-container {
  grid-template-columns: 393px 1fr;
  grid-gap: 30px;
}
.tab .active {
  color: #fff;
  background-color: var(--green);
}
</style>
