<template>
    <div class="account">
        <h1 class="not-visible" data-focus>{{ $t('account.account') }}</h1>

        <div class="account_banner">
            <AUploadArea
                :initial-preview="banner"
                @input="uploadUserBanner"
                :disabled="userAddress !== walletAddress"
                :max-file-size="maxNFTSize"
                :validator="imageValidator"
                strict
            />
        </div>
        <div class="account_header">
            <div class="account_avatar" :class="{ 'account_avatar-noavatar': !avatar }">
                <AUploadArea
                    :initial-preview="avatar"
                    @input="uploadUserAvatar"
                    :disabled="userAddress !== walletAddress"
                    :max-file-size="maxNFTSize"
                    :validator="imageValidator"
                    strict
                    class="auploadarea-nobackground"
                />
                <div v-if="!avatar" v-html="getJazzicon(userAddress, 118)" class="account_avatar_jazzicon"></div>
            </div>
            <div class="account_title">{{ user.username || $t('account.unnamed') }}</div>
            <div class="account_subtitle">
                <f-copy-button :text="userAddress">
                    <template #button-content>
                        <f-ellipsis :text="userAddress" overflow="middle" />
                    </template>
                </f-copy-button>
            </div>
            <div class="account_join"><AccountFollow :user-address="userAddress" /></div>
            <div class="account_btn">
                <AShareButton />
            </div>
        </div>

        <account-navigation ref="accountNavigation" :navigation="navigation" :filters="filters" />

        <!--        <div class="account_filterButton">
            <f-button @click.native="isSideClose = !isSideClose">
                {{ $t('account.filter') }}
                <span v-if="filterNumber" class="account_filterButton_counter">{{ filterNumber }}</span>
            </f-button>
        </div>-->
        <div class="account_content" :class="{ no_aside: isSideClose }">
            <div class="account_sidebar" :class="{ close: isSideClose }">
                <!--                <header class="account_sidebar_header">
                    <div class="account_sidebar_wrap">
                        <button
                            :aria-label="$t('account.filters')"
                            :data-tooltip="$t('account.filters')"
                            @click="isSideClose = !isSideClose"
                        >
                            <span class="account_sidebar_label"
                                ><app-iconset size="24px" icon="filter" />{{ $t('account.filter') }}</span
                            >
                            <span class="account_sidebar_icon"><app-iconset icon="arrowright" size="24px"/></span>
                        </button>
                    </div>
                </header>
                <nft-filters v-model="filters" class="account_sidebar_filters" />-->
            </div>
            <div class="account_view">
                <div class="account_view_filters">
                    <div class="account_view_filters_search">
                        <!--                        <ASearchField field-size="large" placeholder="Search" />-->
                    </div>
                    <div class="account_view_filters_list">
                        <!--                        <NftListFilters v-model="filters" />-->
                        <DensitySwitch v-if="!isPageWithNoFilters" />
                    </div>
                </div>
                <div class="account_view_chips">
                    <NftFilterChips v-model="filters" @chips-change="onChipsChange" />
                </div>
                <section :aria-label="$t('account.accountContent')">
                    <router-view :user-address="userAddress" />
                </section>
            </div>
        </div>
    </div>
</template>

<script>
import AUploadArea from '@/common/components/AUploadArea/AUploadArea';
import AShareButton from '@/common/components/AShareButton/AShareButton';
// import NftFilters from '@/modules/nfts/components/NftFilters/NftFilters.vue';
// import AppIconset from '@/modules/app/components/AppIconset/AppIconset';
// import ASearchField from '@/common/components/ASearchField/ASearchField';
// import NftListFilters from '@/modules/nfts/components/NftListFilters/NftListFilters';
import DensitySwitch from '@/modules/nfts/components/DensitySwitch/DensitySwitch';
import NftFilterChips from '@/modules/nfts/components/NftFilterChips/NftFilterChips.vue';
import { routeQueryMixin } from '@/common/mixins/route-query.js';
import FEllipsis from 'fantom-vue-components/src/components/FEllipsis/FEllipsis.vue';
import FCopyButton from 'fantom-vue-components/src/components/FCopyButton/FCopyButton.vue';
import AccountFollow from '@/modules/account/components/AccountFollow/AccountFollow.vue';
import { mapState } from 'vuex';
import { getUser } from '@/modules/account/queries/user.js';
import AccountNavigation from '@/modules/account/components/AccountNavigation/AccountNavigation.vue';
import { defer } from 'fantom-vue-components/src/utils';
import { getUserTokenCounters } from '@/modules/account/queries/user-token-counters.js';
import { signIn, getBearerToken } from '@/modules/account/auth.js';
import { uploadUserFile } from '@/utils/upload.js';
import { getImageThumbUrl, getIPFSUrl } from '@/utils/url.js';
import { toInt } from '@/utils/big-number.js';
import { getJazzicon } from '@/utils/jazzicon.js';
import { getBannedTokensCount } from '@/modules/account/queries/banned-tokens.js';
import { compareAddresses } from '@/utils/address.js';
import { documentMeta } from '@/modules/app/DocumentMeta.js';
import { focusElem } from 'fantom-vue-components/src/utils/aria.js';
import appConfig from '@/app.config.js';
import { imageValidator } from '@/common/components/AUploadArea/validators.js';

const onlyModeratorRoutes = [
    {
        name: 'account-banned-tokens',
        label: 'account.bannedTokens',
    },
    {
        name: 'account-collections',
        label: 'account.collections',
    },
];

export default {
    name: 'Account',

    mixins: [routeQueryMixin('filters')],

    components: {
        AccountNavigation,
        AccountFollow,
        AUploadArea,
        AShareButton,
        // AppIconset,
        // NftFilters,
        // ASearchField,
        // NftListFilters,
        DensitySwitch,
        NftFilterChips,
        FEllipsis,
        FCopyButton,
    },

    data() {
        return {
            filters: {},
            isSideClose: true,
            filterNumber: 0,
            userAddress: this.$route.params.adddress,
            user: {},
            avatar: null,
            banner: null,
            maxNFTSize: appConfig.settings.maxNFTSize,
            navigation: [
                {
                    routeName: 'account-single-items',
                    label: this.$t('account.collected'),
                    // icon: 'solid1',
                    counter: 0,
                },
                {
                    routeName: 'account-created',
                    label: this.$t('account.created'),
                    // icon: 'paint',
                    counter: 0,
                },
                {
                    routeName: 'account-favorited',
                    label: this.$t('account.favorited'),
                    // icon: 'like',
                    counter: 0,
                },
                /*{
                    routeName: 'account-hidden',
                    label: this.$t('account.hidden'),
                    icon: 'eyeslash',
                    counter: 0,
                },*/
                {
                    routeName: 'account-activity',
                    label: this.$t('account.activity'),
                    // icon: 'history',
                    counter: 0,
                },
                {
                    routeName: 'account-offers',
                    label: this.$t('account.offers'),
                    // icon: 'tag',
                    counter: 0,
                },
                {
                    routeName: 'account-my-offers',
                    label: this.$t('account.myOffers'),
                    //icon: 'tag',
                    counter: 0,
                },
            ],
        };
    },

    computed: {
        ...mapState('wallet', {
            walletAddress: 'account',
            userIsModerator: 'userIsModerator',
        }),

        isPageWithNoFilters() {
            return this.$route.matched.some(
                ({ name }) =>
                    name === 'account-activity' ||
                    name === 'account-offers' ||
                    name === 'account-my-offers' ||
                    name === 'account-collections'
            );
        },

        accountUserIsModerator() {
            return this.userIsModerator && compareAddresses(this.userAddress, this.walletAddress);
        },
    },

    watch: {
        walletAddress: {
            handler(value) {
                if (!this.userAddress) {
                    this.userAddress = this.$route.params.address || value;
                } else {
                    this.userAddress = value;
                }

                if (!this.userAddress) {
                    this.$router.push('/');
                } else {
                    this.update();
                }
            },
            immediate: true,
        },

        $route(value) {
            this.userAddress = value.params.address || this.walletAddress;
            this.update();
        },

        userAddress: {
            handler(value) {
                if (value === this.$route.params.address) {
                    this.navigation = this.navigation.map(item => {
                        return { ...item, routeParams: { address: value } };
                    });
                } else {
                    this.navigation = this.navigation.map(item => {
                        delete item.routeParams;
                        return { ...item };
                    });
                }
            },
            immediate: true,
        },

        userIsModerator: {
            handler() {
                this.updateModeratorTabs();
            },
            immediate: true,
        },
    },

    mounted() {
        focusElem(this.$el);
    },

    methods: {
        update() {
            this.loadUser(this.userAddress);

            defer(() => {
                this.updateTabsCounters();

                if (this.accountUserIsModerator) {
                    this.updateBannedTokensCounter();
                }
            }, 200);
        },

        updateModeratorTabs() {
            const { navigation } = this;
            const { accountUserIsModerator } = this;

            onlyModeratorRoutes.forEach(route => {
                const routeName = route.name;
                const navItemIdx = navigation.findIndex(item => item.routeName === routeName);

                if (accountUserIsModerator) {
                    if (navItemIdx === -1) {
                        const navItem = {
                            routeName,
                            label: this.$t(route.label),
                        };

                        if (routeName !== 'account-collections') {
                            navItem.counter = 0;
                        }

                        navigation.push(navItem);
                    }
                } else {
                    if (navItemIdx > -1) {
                        navigation.splice(navItemIdx, 1);
                    }
                }
            });

            if (accountUserIsModerator) {
                this.updateBannedTokensCounter();
            }
        },

        async checkUserSingIn() {
            let ok = true;
            if (!getBearerToken()) {
                ok = await signIn();
            }

            return ok;
        },

        async uploadUserAvatar(files) {
            let isSignIn = await this.checkUserSingIn();
            if (isSignIn) {
                const uploadOk = await uploadUserFile(files, 'avatar');

                if (uploadOk) {
                    window.location.reload();
                }
            }
        },

        async uploadUserBanner(files) {
            let isSignIn = await this.checkUserSingIn();
            if (isSignIn) {
                uploadUserFile(files, 'banner');
            }
        },

        /**
         * @param {string} userAddress
         * @return {Promise<void>}
         */
        async loadUser(userAddress) {
            if (userAddress) {
                this.user = await getUser(userAddress);
                this.avatar = this.user.avatarThumb ? getImageThumbUrl(this.user.avatarThumb) : '';
                this.banner = this.user.banner ? getIPFSUrl(this.user.banner) : '';

                this.setMetaInfo();
            }
        },

        /**
         * @param {Object} [user]
         */
        setMetaInfo(user = this.user) {
            const sTitle = documentMeta.getSplittedTitle();

            documentMeta.setMetaInfo({
                title: `${sTitle[0]} | ${this.$t('account.account')} ${user.username || user.address}`,
            });
        },

        /**
         * @return {Promise<void>}
         */
        async updateTabsCounters() {
            const { accountNavigation } = this.$refs;
            const tokenCounters = await getUserTokenCounters(this.userAddress);

            if (!tokenCounters) {
                return;
            }

            if (tokenCounters.createdTokens) {
                accountNavigation.updateCounter('account-created', toInt(tokenCounters.createdTokens.totalCount));
            }
            if (tokenCounters.tokenLikes) {
                accountNavigation.updateCounter('account-favorited', toInt(tokenCounters.tokenLikes.totalCount));
            }
            if (tokenCounters.ownerships) {
                accountNavigation.updateCounter('account-single-items', toInt(tokenCounters.ownerships.totalCount));
            }
            if (tokenCounters.activities) {
                accountNavigation.updateCounter('account-activity', toInt(tokenCounters.activities.totalCount));
            }
            if (tokenCounters.offers) {
                accountNavigation.updateCounter('account-offers', toInt(tokenCounters.offers.totalCount));
            }
            if (tokenCounters.myOffers) {
                accountNavigation.updateCounter('account-my-offers', toInt(tokenCounters.myOffers.totalCount));
            }
        },

        /**
         * @return {Promise<void>}
         */
        async updateBannedTokensCounter() {
            const { accountNavigation } = this.$refs;
            const bannedTokenCounters = await getBannedTokensCount(this.userAddress);

            if (!bannedTokenCounters || !accountNavigation) {
                return;
            }

            accountNavigation.updateCounter('account-banned-tokens', toInt(bannedTokenCounters.totalCount));
        },

        /**
         * @param {Array} chips
         */
        onChipsChange(chips) {
            this.filterNumber = chips.length;
        },

        getJazzicon,
        imageValidator,
    },
};
</script>
<style lang="scss">
@use 'style';
</style>
