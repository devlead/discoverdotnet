<template>
    <div>
        <div v-if="title" class="text-center">        
            <h1>
                <span class="accent-secondary">All</span>
                <span class="accent-primary">{{ title }}</span>
            </h1>
            <hr />        
            <h5 class="accent-script">
                {{ filteredCardData.length }} {{ title }}
                <small v-if="filteredCardData.length != cardData.length">(Of {{ cardData.length }} Total)</small>
            </h5>
        </div>

        <a id="card-grid-anchor"></a>

        <b-navbar toggleable="md" variant="dark" type="dark" class="my-4">
            <b-navbar-toggle target="filter-nav-collapse"></b-navbar-toggle>
            <b-collapse is-nav id="filter-nav-collapse">
                <b-navbar-nav>   
                    <span v-if="filters" v-for="(filter, index) in filters" :key="filter.text">
                        <filter-dropdown
                            v-if="'field' in filter || 'values' in filter"
                            :text="filter.text"
                            :values="filterValues(filter)"
                            :selected.sync="selected[index]">
                        </filter-dropdown>
                        <b-nav-item
                            v-else
                            :active="selected[index]"
                            @click="filterItemClicked(index)"
                            href="#">
                                {{ filter.text }}
                        </b-nav-item>
                    </span>
                    <b-nav-item-dropdown v-if="sorts.length > 0" text="Sort">
                        <b-dropdown-item v-for="sort in sorts" :key="sort.text" @click="applySort(sort)">
                            {{ sort.text }}
                        </b-dropdown-item>
                    </b-nav-item-dropdown>
                </b-navbar-nav>
                <b-navbar-nav class="ml-auto">
                    <b-nav-item v-if="suggestLink" :href="suggestLink">Suggest New</b-nav-item>
                    <b-nav-form>
                        <b-button size="sm" class="mr-sm-2 mt-2 mt-sm-0" @click="shuffle">Shuffle The Deck</b-button>
                        <div class="w-100 mb-2 d-lg-none"></div>
                    </b-nav-form>
                </b-navbar-nav>
            </b-collapse>
        </b-navbar>   

        <div class="row">
            <div v-for="cardData in pagedCardData" :key="cardData" :class="columnClasses" class="mb-4 full-height-card">
                <slot :card-data="cardData">
                    <component :is="getCard(cardData)" :card-data="cardData"></component>
                </slot>
            </div>
        </div>
            
        <b-pagination
            v-if="filteredCardData.length > perPage"
            :total-rows="filteredCardData.length"
            v-model="currentPage"
            :per-page="perPage"
            :last-text="'&raquo; ' + filteredCardData.length + ' Total'"
            align="center"
            @input="pageChange">
        </b-pagination>
    </div>
</template>

<script>
    module.exports = {
        props: {
            title: null,
            suggestLink: null,
            cardJson: null,
            filters: null,
            sorts: null,
            perPage:
            {
                default: 24
            },
            columnClasses:
            {                
                default: "col-sm-6 col-md-4 col-lg-3"
            },
            randomizeOrder: null
        },
        data: function() {
            return {
                cardData: [],
                selected: [],
                currentPage: 1
            }
        },
        created: function() {
            var self = this;
            
            if(this.filters) {
                // Prepare the filter selection array
                // See https://stackoverflow.com/q/25512771/807064 
                this.selected = Array
                    .apply(null, new Array(this.filters.length))
                    .map(function(item, index){ 
                        var filter = self.filters[index];
                        if('field' in self.filters[index] || 'values' in self.filters[index])
                        {
                            return new Array(); 
                        } else {
                            return false;
                        }
                    });
            }

            // Get the card data
            axios
                .get(this.cardJson)
                .then(response => {
                    if(this.randomizeOrder) {
                        this.cardData = response.data.sort(function(){return 0.5 - Math.random()});
                    } else {
                        this.cardData = response.data;
                    }
                })
                .catch(e => {
                    console.log(e);
                });
        },
        computed: {
            filteredCardData: function() {
                var selected = this.selected;
                var filtered = this.cardData;
                if(this.filters) {
                    this.filters.forEach(function(filter, index) {
                        if('filter' in filter) {
                            // Call the specified filter function
                            filtered = filter['filter'](filtered, selected[index]);
                        } else {
                            // Filter by the specificed field prop
                            if(selected[index].length > 0) {
                                filtered = filtered.filter(function(item) {
                                    if(filter.field in item) {
                                        if(Array.isArray(item[filter.field])) {
                                            return selected[index].some(function(search) {
                                                return item[filter.field].indexOf(search) !== -1;
                                            });
                                        }
                                        return selected[index].indexOf(item[filter.field]) !== -1;
                                    }
                                    return false;
                                });
                            }
                        }
                    });
                }

                return filtered;
            },
            pagedCardData: function() {
                var filtered = this.filteredCardData;
                
                if(filtered.length > this.perPage) {
                    filtered = filtered.slice((this.currentPage - 1) * this.perPage, this.currentPage * this.perPage);
                }

                return filtered;
            }
        },
        methods: {
            shuffle: function() {
                this.cardData = this.cardData.sort(function(){return 0.5 - Math.random()});
            },
            applySort: function(sortData) {
                if("apply" in sortData) {
                    var self = this;
                    sortData.apply(function() {
                        self.cardData = self.cardData.sort(sortData.sort);
                    });
                } else {
                    this.cardData = this.cardData.sort(sortData.sort);
                }
            },
            filterValues: function(filter) {
                // If raw values were provided, use those
                if('values' in filter) {
                    return filter['values'];
                }

                // Otherwise, map the field values from the data
                return this.cardData.map(function(item) {
                    if(filter.field in item) {
                        return item[filter.field];
                    }
                });
            },
            filterItemClicked: function(index) {
                Vue.set(this.selected, index, !this.selected[index]);
            },
            pageChange: function() {
                Vue.nextTick(function () {
                    document.getElementById('card-grid-anchor').scrollIntoView({ behavior: "smooth" });
                });
            }
        }
    }
</script>