<template>
    <div class="mb-5">
        <div class="text-center mb-5">            
            <h3>
                <span class="accent-secondary">Recent</span>
                <span class="accent-primary">Issues</span>
                
                <!-- This adds extra height to line up the hr -->
                <span class="h1"></span>
            </h3>
            <hr/>
            <p class="small">Some issues from the last 24 hours.</p>            
            <issue-card v-for="issue in firstIssues" :key="issue.link" :card-data="issue" :project-keys="projectKeys">
            </issue-card>
            <div class="text-right"><b-button size="sm" class="mr-sm-2 mt-2 mt-sm-0" @click="shuffle">Shuffle</b-button></div>
        </div>
    </div>
</template>


<script>
    module.exports = {
        props: [
            "projectKeys"
        ],
        data: function() {
            return {
                issues: []
            }
        },
        created: function() {
            axios
                .get('/data/issues/recent/all.json')
                .then(response => {
                    this.issues = response.data
                })
                .catch(e => {
                    console.log(e);
                });
        },
        computed: {
            firstIssues: function() {
                return this.issues.slice(0, 4);
            }
        },
        methods: {
            shuffle: function() {
                this.issues = this.issues.sort(function(){return 0.5 - Math.random()});
            }
        }
    }
</script>