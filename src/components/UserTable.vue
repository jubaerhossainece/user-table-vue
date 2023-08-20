<template>
    <div class="container">

        <div class="row">
            <input class="form-control" v-model="searchQuery" @input="filterData" placeholder="Search here">
            <table class="table table-hover">
                <thead>
                    <tr>
                        <th v-for="key in tableHeaders" :key="key" @click="sortData(key)">
                            {{ key }}
                        </th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(user, index) in paginatedData" :key="index">
                        <td v-for="key in tableHeaders" :key="key">{{ user[key] }}</td>
                    </tr>
                </tbody>
            </table>
            <div>
                <button @click="paginate(-1)" :disabled="currentPage === 0">Previous</button>
                <button @click="paginate(1)" :disabled="currentPage === totalPages - 1">Next</button>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            users: [],
            flattenedData: [],
            currentPage: 0,
            itemsPerPage: 10,
            searchQuery: '',
            sortKey: '',
            sortOrder: 'asc',
            }
    },
    computed: {
        tableHeaders() {
            return this.flattenedData.length > 0 ? Object.keys(this.flattenedData[0]) : [];
        },
        totalPages() {
            return Math.ceil(this.flattenedData.length / this.itemsPerPage);
        },
        paginatedData() {
            const startIndex = this.currentPage * this.itemsPerPage;
            const endIndex = startIndex + this.itemsPerPage;

            return this.flattenedData.slice(startIndex, endIndex)
        }
    },
    methods: {
        fetchData() {
            fetch('https://randomuser.me/api/?results=50')
                .then(response => response.json())
                .then(data => {
                    this.users = data.results;
                    this.flattenedData = this.users.map(user => this.flattenedUserData(user));
                });
        },
        flattenedUserData(user) {
            const flattenedUser = {};
            for (const key in user) {
                
                if (typeof user[key] == 'object') {

                    for (const innerkey in user[key]) {

                        if(typeof user[key][innerkey] == 'object'){
                            for(const nestedInnerKey in user[key][innerkey]){
                                flattenedUser[nestedInnerKey] = user[key][innerkey][nestedInnerKey];
                            }
                        }else{
                            flattenedUser[innerkey] = user[key][innerkey];
                        }
                    }
                } else {
                    flattenedUser[key] = user[key];
                }
            }
            return flattenedUser;
        },
        paginate(direction) {
            this.currentPage += direction;
            console.log("currentPage:"+this.currentPage);
        },
        sortData(key) {
            if (this.sortKey === key) {
                this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
            } else {
                this.sortKey = key;
                this.srtOrder = 'asc';
            }
            let data = this.flattenedData;
            
            this.flattenedData = data.sort((a, b) => {
                const aValue = a[key];
                const bValue = b[key];

                if (this.sortOrder == 'asc') {
                    if(!isNaN(aValue) && !isNaN(bValue)){
                        return aValue - bValue;
                    }
                    return aValue.localeCompare(bValue);
                } else {
                    if(!isNaN(aValue) && !isNaN(bValue)){
                        return bValue - aValue;
                    }
                    return bValue.localeCompare(aValue);
                }
            });
        },
        filterData() {
            if (this.searchQuery) {
                this.currentPage = 0;
                const filteredData = this.flattenedData.filter(user => {
                    for (const key in user) {
                        if (user[key].toString().includes(this.searchQuery)) {
                            return true;
                        }
                    }
                    return false;
                });
                this.flattenedData = filteredData.slice(0, this.itemsPerPage);
            } else {
                this.fetchData();
            }
        }
    },
    mounted() {
        this.fetchData();
    }
}
</script>