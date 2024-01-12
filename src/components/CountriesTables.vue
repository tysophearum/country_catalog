<script>
import Fuse from 'fuse.js';
export default {
    data() {
        return {
            searchQuery: "",
            data: [],
            results: [],
            current: [],
            fuse: null,
            itemsPerPage: 25,
            currentPage: 1,
            totalPages: 0,
            asc: null
        };
    },
    mounted() {
        this.fetchData()
    },
    watch: {
        searchQuery: function () {
            this.handleSearch()
        }
    },
    methods: {
        sortAscName: function () {
            this.results.sort((a, b) => a.name.official.localeCompare(b.name.official));
            this.paginatedItems()
            this.asc = true
        },
        sortDescName: function () {
            this.results.sort((a, b) => b.name.official.localeCompare(a.name.official));
            this.paginatedItems()
            this.asc = false
        },
        prevPage: function () {
            if (this.currentPage > 1) {
                this.currentPage--;
                this.paginatedItems()
            }
        },
        nextPage: function () {
            if (this.currentPage < this.totalPages) {
                this.currentPage++;
                this.paginatedItems()
            }
        },
        goToPage: function (page) {
            this.currentPage = page;
            this.paginatedItems()
        },
        paginatedItems: function () {
            const startIndex = (this.currentPage - 1) * this.itemsPerPage;
            const endIndex = startIndex + this.itemsPerPage;
            this.current = this.results.slice(startIndex, endIndex);
        },
        fetchData: function () {
            fetch('https://restcountries.com/v3.1/all')
                .then(response => response.json())
                .then(data => this.data = data)
                .then(() => {
                    this.results = JSON.parse(JSON.stringify(this.data));
                    this.sortAscName()
                    this.fuse = new Fuse(this.data, {
                        keys: ['name.official'], // Adjust the path to the nested property
                        threshold: 0.3, // Adjust the threshold as needed
                    })
                    this.totalPages = Math.ceil(this.results.length / this.itemsPerPage);
                    this.paginatedItems()
                })
        },
        handleSearch: function () {
            if (this.searchQuery != "") {
                this.results = this.fuse.search(this.searchQuery).map(result => result.item)
                this.totalPages = Math.ceil(this.results.length / this.itemsPerPage);
                this.currentPage = 1
                this.paginatedItems()
            } else if (this.searchQuery == "") {
                this.results = JSON.parse(JSON.stringify(this.data));
                this.sortAscName()
                this.totalPages = Math.ceil(this.results.length / this.itemsPerPage);
            }
        }
    }

}
</script>
<template>
    <div class=" w-[95vw] min-w-[1000px] overflow-scroll">
        <input v-on:change="search" v-model="searchQuery" type="search" class="bg-purple-white shadow rounded border-0 p-3 my-2 w-1/3" placeholder="Search by name...">
        <div class="">
            <div class="bg-blue-600 text-white grid grid-cols-[10%,14%,14%,14%,18%,14%,14%] rounded">
                <div scope="col" class="px-6 py-3 whitespace-nowrap">
                    Flag
                </div>
                <div scope="col" class="px-6 py-3 whitespace-nowrap">
                    <div>
                        <span class="mr-2">Country name</span>
                        <button v-if="asc == true" @click="sortDescName">▲</button>
                        <button v-else-if="asc == false" @click="sortAscName">▼</button>
                    </div>
                </div>
                <div scope="col" class="px-6 py-3 whitespace-nowrap">
                    Country Code 2C
                </div>
                <div scope="col" class="px-6 py-3 whitespace-nowrap">
                    Country Code 3C
                </div>
                <div scope="col" class="px-6 py-3 whitespace-nowrap">
                    Native Country Name
                </div>
                <div scope="col" class="px-6 py-3 whitespace-nowrap">
                    Alt Country Name
                </div>
                <div scope="col" class="px-6 py-3 whitespace-nowrap">
                    Country Calling Codes
                </div>
            </div>
            <div class="h-[72vh] overflow-scroll mt-2">
                <div v-for="(item, index) in current" :key="index"
                    class="bg-white rounded-sm shadow my-1  grid grid-cols-[10%,14%,14%,13%,18%,14%,14%] gap-1 duration-150 border hover:border hover:border-blue-600 hover:bg-blue-50">
                    <div scope="row"
                        class="flex items-center px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white text-4xl">
                        {{ item.flag }}
                    </div>
                    <div class="flex flex-col justify-center px-6 py-4">
                        <button class="text-start" :onclick="`my_modal_${index}.showModal()`">{{ item.name.official }}</button>
                        <dialog :id="`my_modal_${index}`" class="modal">
                            <div class="modal-box w-fit max-w-5xl">
                                <div class="flex">
                                    <div class=" flex flex-col justify-center">
                                        <div class=" border border-gray-500 p-3">
                                            <img class="min-w-96 w-[120%]" :src="item.flags.png" alt="">
                                        </div>
                                        <h1 class=" text-xl font-bold flex items-center my-3">
                                            {{ item.name.official }}
                                            <span v-if="item.independent"
                                                class="ml-2 text-xs text-white bg-green-700 rounded-xl w-fit px-1 py-0.5">Independent</span>
                                            <span v-else
                                                class="ml-2 text-xs text-white bg-red-700 rounded-xl w-fit px-1 py-0.5">Not
                                                Independent</span>
                                        </h1>
                                        <div v-if="item.capital"><span class="font-bold">Capital</span> : {{ item.capital[0]
                                        }}</div>
                                    </div>
                                    <div class="border-l-2 ml-3 px-2">
                                        <h2 class="text-xl font-bold">More Information</h2>
                                        <div class="my-2" v-if="item.continents">Continent: {{ item.continents[0] }}</div>
                                        <div class="my-2">Population: {{ new
                                            Intl.NumberFormat('en-US').format(item.population) }}</div>
                                        <div class="my-2" v-if="item.status">Status: {{ item.status }}</div>
                                        <div class="flex justify-between my-2">
                                            <div class="mr-2 whitespace-nowrap">Currency use: </div>
                                            <div v-if="item.currencies"
                                                :class="`${Object.values(item.currencies).length > 1 ? 'max-h-24 w-[70%] overflow-scroll border-2 border-blue-800 rounded-lg p-3' : ''}`">
                                                <div class=" whitespace-nowrap border my-0.5 border-blue-700 bg-white px-2 py-1 rounded-lg"
                                                    v-if="item.currencies" v-for="cur in Object.values(item.currencies)">{{
                                                        cur.name }} ({{ cur.symbol }})</div>
                                                <div v-else class=" text-gray-700">Unknown</div>
                                            </div>
                                        </div>
                                        <div class="flex justify-between my-2">
                                            <div class="mr-2" v-if="item.borders">Borders: </div>
                                            <div v-if="item.borders"
                                                :class="`${item.borders.length > 1 ? 'max-h-24 w-[70%] overflow-scroll border-2 border-blue-800 rounded-lg p-3' : ''}`">
                                                <div class=" whitespace-nowrap border my-0.5 border-blue-700 bg-white px-2 py-1 rounded-lg"
                                                    v-for="time in item.borders">{{ time }}</div>
                                            </div>
                                        </div>
                                        <div class="flex justify-between my-2">
                                            <div class="mr-2" v-if="item.timezones">Timezones: </div>
                                            <div
                                                :class="`${item.timezones.length > 1 ? 'max-h-24 overflow-scroll border-2 border-blue-800 rounded-lg p-3 w-[70%]' : ''}`">
                                                <div class=" whitespace-nowrap border my-0.5 border-blue-700 bg-white px-2 py-1 rounded-lg"
                                                    v-for="time in item.timezones">{{ time }}</div>
                                            </div>
                                        </div>
                                        <div class="flex justify-between my-2">
                                            <div class="mr-2">Translations: </div>
                                            <div class=" max-h-32 overflow-scroll border-2 border-blue-800 rounded-lg p-3 w-[70%]"
                                                v-if="item.translations">
                                                <div class=" whitespace-nowrap border my-0.5 border-blue-700 bg-white px-2 py-1 rounded-lg"
                                                    v-for="tran in Object.entries(item.translations)">{{ tran[0] }}: {{
                                                        tran[1].official }}</div>
                                            </div>
                                        </div>
                                        <div class="my-2">United Nation member: {{ item.unMember }}</div>
                                    </div>
                                </div>
                                <div class="modal-action">
                                    <form method="dialog">
                                        <!-- if there is a button, it will close the modal -->
                                        <button class="btn">Close</button>
                                    </form>
                                </div>
                            </div>
                        </dialog>
                    </div>
                    <div class="flex flex-col justify-center px-6 py-4">
                        {{ item.cca2 }}
                    </div>
                    <div class="flex flex-col justify-center px-6 py-4">
                        {{ item.cca3 }}
                    </div>
                    <div class="flex flex-col justify-center px-6 py-4">
                        <div v-if="item.name.nativeName">
                            <div class="flex">
                                <div class=" ">{{ Object.entries(item.name.nativeName)[0][0] }}:</div>
                                <div class="w-1"></div>
                                <div>{{ Object.entries(item.name.nativeName)[0][1].official }}</div><br>
                            </div>
                            <details v-if="Object.entries(item.name.nativeName).length > 1" class="dropdown">
                                <summary
                                    class="border cursor-pointer w-fit h-fit rounded text-sm px-1 bg-blue-600 text-gray-100">
                                    {{ Object.entries(item.name.nativeName).length - 1 }} more</summary>
                                <ul class="p-2 shadow-lg dropdown-content z-50 bg-base-100 rounded-box w-fit ">
                                    <li class="p-2 border-b" v-for="name in Object.entries(item.name.nativeName).slice(1)">
                                        <a class=" whitespace-nowrap">{{ name[0] }}: {{ name[1].official }}</a>
                                    </li>
                                </ul>
                            </details>
                        </div>
                    </div>
                    <div class="flex flex-col justify-center px-6 py-4">
                        <div class="my-1 whitespace-nowrap" v-if="item.altSpellings">
                            {{ item.altSpellings[0] }}
                        </div>
                        <details v-if="item.altSpellings.length > 1" class="dropdown ">
                            <summary
                                class="border cursor-pointer w-fit h-fit rounded text-sm px-1 bg-blue-600 text-gray-100">{{
                                    item.altSpellings.length - 1 }} more</summary>
                            <ul class="p-2 shadow-lg dropdown-content z-50 bg-base-100 rounded-box w-fit">
                                <li class="p-2 border-b" v-for="spell in item.altSpellings.slice(1)"><a
                                        class=" whitespace-nowrap">{{ spell }}</a></li>
                            </ul>
                        </details>
                    </div>
                    <div class="flex flex-col justify-center px-6 py-4">
                        <div v-if="item.idd.suffixes">
                            {{ item.idd.root }}
                            {{ item.idd.suffixes[0] }} <br>
                            <details v-if="item.idd.suffixes.length > 1" class="dropdown">
                                <summary
                                    class="border cursor-pointer w-fit h-fit rounded text-sm px-1 bg-blue-600 text-gray-100">
                                    {{ item.idd.suffixes.length - 1 }} more</summary>
                                <ul
                                    class="p-2 shadow-lg dropdown-content z-50 bg-base-100 rounded-box w-fit max-h-80 overflow-scroll">
                                    <li class="py-2 px-7 border-b" v-for="num in item.idd.suffixes.slice(1)"><a
                                            class=" whitespace-nowrap">{{ item.idd.root }} {{ num }}</a></li>
                                </ul>
                            </details>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="flex justify-center w-full">
            <div class="w-fit mt-3">
                <button @click="prevPage" :disabled="currentPage === 1" class=" text-blue-600 tracking-[0.2rem]">←
                    PREV</button>
                <button v-for="pag in Array.from({ length: totalPages }, (_, index) => index + 1)" @click="goToPage(pag)"
                    :class="`${pag == currentPage ? 'bg-blue-900' : 'bg-blue-600'} btn btn-circle mx-0.5 text-white hover:bg-blue-800`">{{
                        pag }}</button>
                <!-- <span>{{ currentPage }}</span> -->
                <button @click="nextPage" :disabled="currentPage === totalPages"
                    class=" text-blue-600 tracking-[0.2rem]">NEXT →</button>
            </div>
        </div>
</div></template>

<style scoped></style>