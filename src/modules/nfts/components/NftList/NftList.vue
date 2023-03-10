<template>
    <f-infinite-scroll
        ref="infScroll"
        :total-items="totalItems"
        :per-page="perPage"
        :curr-page="currPage"
        :has-next-page="hasNextPage"
        :loading="loading"
        :root="root"
        :root-margin="rootMargin"
        v-on="$listeners"
    >
        <div class="nftlist" :class="[`nftlist-density-${cDensity}`, { 'scrollsnap-horizontal': horizontal }]">
            <nft-card
                v-for="nft in tokens"
                :nftData="nft"
                :key="`${nft.contract}_${nft.tokenId}`"
                :data-page="nft._page || null"
                @mousedown.native="onNftMousedown"
                @token-ban-unban="onTokenBanUnban"
            />
        </div>
    </f-infinite-scroll>
</template>
<script>
import NftCard from '@/modules/nfts/components/NftCard/NftCard.vue';
import FInfiniteScroll from 'fantom-vue-components/src/components/FInfiniteScroll/FInfiniteScroll.vue';
import FPagination from 'fantom-vue-components/src/components/FPagination/FPagination.vue';
import FIntersectionObserver from 'fantom-vue-components/src/components/FIntersectionObserver/FIntersectionObserver.vue';

export default {
    name: 'NftList',

    components: { NftCard, FInfiniteScroll },

    props: {
        tokens: {
            type: Array,
            default() {
                return [];
            },
        },
        density: {
            type: [Number, String],
            default: 'large',
        },
        loading: {
            type: Boolean,
            default: false,
        },
        /** Horizontal layout */
        horizontal: {
            type: Boolean,
            default: false,
        },
        hasNextPage: {
            type: Boolean,
            default: true,
        },
        /** Total amount of items (FPagination prop) */
        totalItems: { ...FPagination.props.totalItems },
        /** Number of items per page (FPagination prop) */
        perPage: { ...FPagination.props.perPage },
        /** Current page number (FPagination prop) */
        currPage: { ...FPagination.props.currPage },
        ...FIntersectionObserver.props,
    },

    data() {
        return {
            likedNftIds: [],
        };
    },

    computed: {
        cDensity() {
            const { density } = this;
            const densityNum = parseInt(density);

            if (!isNaN(densityNum)) {
                return densityNum === 200 ? 'small' : 'large';
            }

            return density;
        },
    },

    methods: {
        /**
         * @param {number} pageNum
         */
        goToPageNum(pageNum) {
            const { infScroll } = this.$refs;

            if (infScroll) {
                infScroll.goToPage(pageNum, true);
            }
        },

        /**
         * @param {{contract: string, tokenId: string}} token
         * @return {*}
         */
        findTokenIndex(token) {
            return this.tokens.findIndex(item => item.contract === token.contract && item.tokenId === token.tokenId);
        },

        onNftMousedown(event) {
            const nftElem = event.target.closest('[data-page]');
            const page = nftElem ? parseInt(nftElem.getAttribute('data-page')) : -1;

            if (page > -1) {
                this.$emit('nft-list-page', page);
            }
        },

        /**
         * @param {{token: Object, ban?: boolean, unban?: boolean}} payload
         */
        onTokenBanUnban(payload) {
            const idx = this.findTokenIndex(payload.token);

            if (idx > -1) {
                this.tokens.splice(idx, 1);
            }
        },
    },
};
</script>

<style lang="scss">
@use 'style';
</style>
