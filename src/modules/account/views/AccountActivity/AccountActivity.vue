<template>
    <div class="accountactivity">
        <h2 class="not-visible" data-focus>{{ $t('page.accountActivity.title') }}</h2>

        <NftItemActivityFilter v-model="filters" />
        <NftItemActivityFilterChips v-show="filters.length > 0" v-model="filters" />

        <account-activity-grid :filters="filters" :user-address="userAddress" />
    </div>
</template>

<script>
import AccountActivityGrid from '@/modules/account/components/AccountActivityGrid/AccountActivityGrid.vue';
import { focusElem } from 'fantom-vue-components/src/utils/aria.js';
import NftItemActivityFilter from '@/modules/nfts/components/NftItemActivityFilter/NftItemActivityFilter';
import NftItemActivityFilterChips from '@/modules/nfts/components/NftItemActivityFilterChips/NftItemActivityFilterChips';

export default {
    name: 'AccountActivity',

    components: { AccountActivityGrid, NftItemActivityFilter, NftItemActivityFilterChips },

    props: {
        userAddress: {
            type: String,
            default: '',
            required: true,
        },
    },

    data() {
        return {
            /** Values from ITEM_ACTIVITY_FILTER_OPTIONS */
            filters: [],
        };
    },

    mounted() {
        focusElem(this.$el);
    },
};
</script>

<style lang="scss">
.accountactivity {
    display: flex;
    flex-direction: column;
    gap: var(--f-spacer-3);

    .accountactivityfilter {
        max-width: 320px;
    }
}
</style>
