<template>
    <div>
        <feed-item-card v-for="card in cardData" :key="card.link" :icon="icon" :card-data="card">
        </feed-item-card>
    </div>
</template>

<script>
    module.exports = {
        props: [
            'feedItemJson',
            'icon'
        ],
        data: function() {
            return {
                cardData: []
            }
        },
        created: function() {
            axios
                .get(this.feedItemJson)
                .then(response => {
                    this.cardData = response.data
                })
                .catch(e => {
                    console.log(e);
                });
        }
    }
</script>