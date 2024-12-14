<script setup>
    import { ref, reactive, onMounted } from 'vue'
    import axios from 'axios';
    import Sidebar from '@/components/Sidebar.vue';
    import Header from '@/components/Header.vue';
    import GitHubRepoCard from '@/components/skeleton/GitHubRepoCard.vue';

    const gitHubProfile = ref([])
    const gitHubRepos = ref([])
    const gitHubPaginatedRepos = ref([])
    const perPage = ref(9)
    const page = ref(1)
    const isFetchingGitHubRepos = ref(true)


    const chartAreaOptions = reactive({
      chart: {
        id: 'area-chart',
        toolbar: {
          show: false,
        },
      },
      xaxis: {
        categories: []
      },
      stroke: {
        curve: 'smooth'
      },
      markers: {
        size: 5
      }
    });

    const chartAreaSeries = ref([]);

    const chartBarOptions = reactive({
        chart: {
          id: 'bar-chart',
          toolbar: {
            show: false,
          },
        },
        xaxis: {
          categories: []
        },
        stroke: {
          curve: 'smooth'
        },
        markers: {
          size: 5
        }
      });

    const chartBarSeries = ref([]);

    const fetchGitHubProfile = () => {
      return axios.get('');
    };

    const fetchGitHubRepos = () => {
      return axios.get('/repos');
    };

    const fetchGitHubPaginatedRepos = () => {
      return axios.get(`/repos?per_page=${perPage.value}&page=${page.value}`);
    };

    const refreshGitHubRepos = async () => {
        try {
          const response = await fetchGitHubPaginatedRepos()
          gitHubPaginatedRepos.value = response.data
          isFetchingGitHubRepos.value = false
        } catch (error) {
          alert(`Failed to fetch tasks: ${error}`)
        }
    }

    const getChartSeries = (gitHubRepos) => {
      return [
        {
          name: 'Stars',
          data: gitHubRepos.map(repo => repo.stargazers_count),
          color: "#f05252",
        },
        {
          name: 'Forks',
          data: gitHubRepos.map(repo => repo.forks_count),
          color: "#1A56DB",
        },
        {
          name: 'Watchers',
          data: gitHubRepos.map(repo => repo.watchers_count),
          color: "#7E3BF2",
        }
      ]
    }

    const setCharts = (gitHubRepos) => {
      chartAreaOptions.xaxis.categories = gitHubRepos.map(repo => repo.name)
      chartAreaSeries.value = getChartSeries(gitHubRepos)

      chartBarOptions.xaxis.categories = gitHubRepos.map(repo => repo.name)
      chartBarSeries.value = getChartSeries(gitHubRepos)
    }

    const prevPage = () => {
      page.value--
      isFetchingGitHubRepos.value = true
      refreshGitHubRepos()
    }

    const nextPage = () => {
      page.value++
      isFetchingGitHubRepos.value = true
      refreshGitHubRepos()
    }

    onMounted(async () => {
        try {
          const [gitHubProfileResponse, gitHubReposResponse, gitHubPaginatedReposResponse] = await Promise.all([
                  fetchGitHubProfile(),
                  fetchGitHubRepos(),
                  fetchGitHubPaginatedRepos(),
              ]);

            gitHubProfile.value = gitHubProfileResponse.data
            gitHubRepos.value = gitHubReposResponse.data
            gitHubPaginatedRepos.value = gitHubPaginatedReposResponse.data
            isFetchingGitHubRepos.value = false

            setCharts(gitHubReposResponse.data)
        } catch (error) {
            alert(`Error during fetching data: ${error}`)
        }
    });
</script>

<template>
    <Header :gitHubProfile="gitHubProfile"></Header>
    <Sidebar></Sidebar>
    <div class="p-4 sm:ml-64">
        <div class="p-4 rounded-lg dark:border-gray-700 mt-14">
            <div class="grid grid-cols-1 gap-4 sm:grid-cols-2 md:gap-6 xl:grid-cols-3 2xl:gap-7.5 mb-4">
                <div class="rounded-sm border border-stroke bg-white p-4 shadow-default dark:border-strokedark dark:bg-boxdark md:p-6 xl:p-7.5">
                  <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-folder-git"><circle cx="12" cy="13" r="2"/><path d="M20 20a2 2 0 0 0 2-2V8a2 2 0 0 0-2-2h-7.9a2 2 0 0 1-1.69-.9L9.6 3.9A2 2 0 0 0 7.93 3H4a2 2 0 0 0-2 2v13a2 2 0 0 0 2 2Z"/><path d="M14 13h3"/><path d="M7 13h3"/></svg>
                  <h4 class="mb-2 mt-5 font-medium">Public Repos</h4>
                  <h3 class="mb-2 text-title-md font-bold text-black dark:text-white">
                    {{ gitHubProfile.public_repos }}
                  </h3>
                  <p class="flex items-center gap-1 text-sm font-medium">
                    <svg width="14" height="14" viewBox="0 0 14 14" fill="none" xmlns="http://www.w3.org/2000/svg">
                      <path d="M13.0158 4.74683H9.4939C9.2314 4.74683 9.01265 4.96558 9.01265 5.22808C9.01265 5.49058 9.2314 5.70933 9.4939 5.70933H11.6595L8.85953 7.59058C8.75015 7.67808 8.59703 7.67808 8.46578 7.59058L5.57828 5.68745C5.1189 5.3812 4.55015 5.3812 4.09078 5.68745L0.722027 7.94058C0.503277 8.0937 0.437652 8.39995 0.590777 8.6187C0.678277 8.74995 0.831402 8.83745 1.0064 8.83745C1.0939 8.83745 1.20328 8.81558 1.2689 8.74995L4.65953 6.49683C4.7689 6.40933 4.92203 6.40933 5.05328 6.49683L7.94078 8.42183C8.40015 8.72808 8.9689 8.72808 9.42828 8.42183L12.5127 6.3437V8.77183C12.5127 9.03433 12.7314 9.25308 12.9939 9.25308C13.2564 9.25308 13.4752 9.03433 13.4752 8.77183V5.22808C13.5189 4.96558 13.2783 4.74683 13.0158 4.74683Z" fill="#10B981"></path>
                    </svg>
                    <span class="text-meta-3">+2.5%</span>
                    <span>than last Week</span>
                  </p>
                </div>
                <div class="rounded-sm border border-stroke bg-white p-4 shadow-default dark:border-strokedark dark:bg-boxdark md:p-6 xl:p-7.5">
                  <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-rss"><path d="M4 11a9 9 0 0 1 9 9"/><path d="M4 4a16 16 0 0 1 16 16"/><circle cx="5" cy="19" r="1"/></svg>

                  <h4 class="mb-2 mt-5 font-medium">Followers</h4>
                  <h3 class="mb-2 text-title-md font-bold text-black dark:text-white">
                    {{ gitHubProfile.followers }}
                  </h3>
                  <p class="flex items-center gap-1 text-sm font-medium">
                    <svg width="14" height="14" viewBox="0 0 14 14" fill="none" xmlns="http://www.w3.org/2000/svg">
                      <path d="M13.0157 4.74683C12.7532 4.74683 12.5344 4.96558 12.5344 5.22808V7.6562L9.4063 5.57808C8.94693 5.27183 8.37818 5.27183 7.9188 5.57808L5.0313 7.50308C4.92193 7.59058 4.7688 7.59058 4.63755 7.50308L1.24693 5.24995C1.02818 5.09683 0.721929 5.16245 0.568804 5.3812C0.415679 5.59995 0.481304 5.9062 0.700054 6.05933L4.09068 8.31245C4.55005 8.6187 5.1188 8.6187 5.57818 8.31245L8.46568 6.38745C8.57505 6.29995 8.72818 6.29995 8.85943 6.38745L11.6594 8.2687H9.49381C9.23131 8.2687 9.01255 8.48745 9.01255 8.74995C9.01255 9.01245 9.23131 9.2312 9.49381 9.2312H13.0157C13.2782 9.2312 13.4969 9.01245 13.4969 8.74995V5.22808C13.5188 4.96558 13.2782 4.74683 13.0157 4.74683Z" fill="#FB5454"></path>
                    </svg>
                    <span class="text-red">-1.5%</span>
                    <span>than last Week</span>
                  </p>
                </div>
                <div class="rounded-sm border border-stroke bg-white p-4 shadow-default dark:border-strokedark dark:bg-boxdark md:p-6 xl:p-7.5">
                  <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-user-check"><path d="M16 21v-2a4 4 0 0 0-4-4H6a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><polyline points="16 11 18 13 22 9"/></svg>

                  <h4 class="mb-2 mt-5 font-medium">Following</h4>
                  <h3 class="mb-2 text-title-md font-bold text-black dark:text-white">
                    {{ gitHubProfile.following }}
                  </h3>
                  <p class="flex items-center gap-1 text-sm font-medium">
                    <svg width="14" height="14" viewBox="0 0 14 14" fill="none" xmlns="http://www.w3.org/2000/svg">
                      <path d="M13.0155 4.74683H9.49366C9.23116 4.74683 9.01241 4.96558 9.01241 5.22808C9.01241 5.49058 9.23116 5.70933 9.49366 5.70933H11.6593L8.85928 7.59058C8.74991 7.67808 8.59678 7.67808 8.46553 7.59058L5.57803 5.68745C5.11866 5.3812 4.54991 5.3812 4.09053 5.68745L0.721783 7.94058C0.503033 8.0937 0.437408 8.39995 0.590533 8.6187C0.678033 8.74995 0.831157 8.83745 1.00616 8.83745C1.09366 8.83745 1.20303 8.81558 1.26866 8.74995L4.65928 6.49683C4.76866 6.40933 4.92178 6.40933 5.05303 6.49683L7.94053 8.42183C8.39991 8.72808 8.96866 8.72808 9.42803 8.42183L12.5124 6.3437V8.77183C12.5124 9.03433 12.7312 9.25308 12.9937 9.25308C13.2562 9.25308 13.4749 9.03433 13.4749 8.77183V5.22808C13.5187 4.96558 13.278 4.74683 13.0155 4.74683Z" fill="#10B981"></path>
                    </svg>

                    <span class="text-meta-3">+2.6%</span>
                    <span>than last Week</span>
                  </p>
                </div>
            </div>
            <div class="col-span-12 mb-4">
              <div class="rounded-sm border border-stroke bg-white shadow-default dark:border-strokedark dark:bg-boxdark">
                <div class="p-4 md:p-6 xl:p-7.5">
                    <div class="flex items-start justify-between">
                      <div>
                        <h2 class="text-title-sm2 font-bold text-black dark:text-white">
                          Repositories
                        </h2>
                      </div>
                    </div>
                </div>
                <div class="bg-white p-6">
                  <!-- Repository Grid -->
                  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                    <!-- Repository Card -->
                     <template v-if="isFetchingGitHubRepos">
                      <template v-for="skeleton in perPage">
                        <GitHubRepoCard></GitHubRepoCard>
                      </template>
                    </template>
                    <div v-if="! isFetchingGitHubRepos" v-for="gitHubRepo in gitHubPaginatedRepos" :key="gitHubRepo.id" class="p-4 border rounded-md shadow-sm bg-gray-50">
                      <div class="flex items-center justify-between">
                        <h3 class="text-blue-600 font-semibold">
                          {{ gitHubRepo.name }}
                        </h3>
                        <span class="text-gray-400 text-xs border px-2 py-0.5 rounded">{{ gitHubRepo.visibility }}</span>
                      </div>
                      <div class="flex items-center mt-2">
                        <div class="flex items-center space-x-2 mr-3">
                          <span class="h-3 w-3 rounded-full bg-red-500"></span>
                          <span class="text-gray-500 text-sm">{{ gitHubRepo.language }}</span>
                        </div>
                        <div class="flex items-center justify-between">
                          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="lucide lucide-star"><path d="M11.525 2.295a.53.53 0 0 1 .95 0l2.31 4.679a2.123 2.123 0 0 0 1.595 1.16l5.166.756a.53.53 0 0 1 .294.904l-3.736 3.638a2.123 2.123 0 0 0-.611 1.878l.882 5.14a.53.53 0 0 1-.771.56l-4.618-2.428a2.122 2.122 0 0 0-1.973 0L6.396 21.01a.53.53 0 0 1-.77-.56l.881-5.139a2.122 2.122 0 0 0-.611-1.879L2.16 9.795a.53.53 0 0 1 .294-.906l5.165-.755a2.122 2.122 0 0 0 1.597-1.16z"/></svg>
                          <div class="text-gray-500 text-sm ml-1">
                            <i class="fas fa-star"></i> {{ gitHubRepo.stargazers_count }}
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>

                  <!-- Pagination Section -->
                  <div class="mt-6 flex justify-end">
                    <button :disabled="page === 1" @click="prevPage"
                      class="px-4 py-2 bg-gray-100 text-gray-700 border rounded-l-md hover:bg-gray-200"
                    >
                      Previous
                    </button>
                    <button :disabled="gitHubPaginatedRepos.length < 9" @click="nextPage"
                      class="px-4 py-2 bg-gray-100 text-gray-700 border rounded-r-md hover:bg-gray-200"
                    >
                      Next
                    </button>
                  </div>
                </div>
              </div>
            </div>
            <div class="grid grid-cols-12 gap-4 md:gap-6 2xl:gap-7.5 mb-4">
                <div class="col-span-12 xl:col-span-7">
                  <div class="col-span-12 rounded-sm border border-stroke bg-white px-5 pb-5 pt-7.5 shadow-default dark:border-strokedark dark:bg-boxdark sm:px-7.5 xl:col-span-8">
                    <div class="w-full bg-white rounded-lg dark:bg-gray-800 p-4 md:p-6">
                      <div class="flex justify-between mb-5">
                        <div>
                          <h5 class="leading-none font-bold text-gray-900 dark:text-white pb-2">Area Chart</h5>
                        </div>
                      </div>
                      <apexchart width="100%" height="315" type="area" :options="chartAreaOptions" :series="chartAreaSeries"></apexchart>
                    </div>
                  </div>
                </div>
                <div class="col-span-12 xl:col-span-5">
                  <div class="rounded-sm border border-stroke bg-white px-5 pb-5 pt-7.5 shadow-default dark:border-strokedark dark:bg-boxdark sm:px-7.5">
                    <div class="w-full bg-white rounded-lg dark:bg-gray-800 p-4 md:p-6">
                      <div class="flex justify-between pb-4 mb-4 border-b border-gray-200 dark:border-gray-700">
                        <div class="flex items-center">
                          <div>
                            <h5 class="leading-none font-bold text-gray-900 dark:text-white pb-2">Graph Chart</h5>
                          </div>
                        </div>
                      </div>
                      <apexchart width="100%" height="300" type="bar" :options="chartBarOptions" :series="chartBarSeries"></apexchart>
                    </div>
                  </div>
                </div>
            </div>
        </div>
    </div>
</template>
  