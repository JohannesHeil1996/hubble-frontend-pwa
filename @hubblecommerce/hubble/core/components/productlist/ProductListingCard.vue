<template>
    <div class="product-card">
        <a :href="routeUrlPds" @click.ctrl.exact="gtmProductClick()" @click.exact.prevent="openDetailPage()">
            <div class="card-media">
                <div class="actions">
                    <div class="badge-wrp">
                        <div v-if="itemIsSpecial" class="badge sale">
                             {{ $t('Sale') }}
                        </div>

                        <div v-if="itemIsNew" class="badge new">
                            {{ $t('New') }}
                        </div>
                    </div>
                </div>

                <img-lazy v-if="!isSlider" class="img-listing" :src="routeUrlProductImg" :alt-info="itemData.name" :title-info="itemData.name" />
                <img v-if="isSlider" :src="routeUrlProductImg" :alt="itemData.name" :title="itemData.name" />
            </div>

            <div class="product-card-info-wrp-link">
                <div class="product-card-info-wrp">
                    <div v-if="itemData.name !== null" class="product-name" v-text="itemData.name" />

                    <div :class="classesExcl" class="price-box price-excluding-tax product-price">
                        <template v-if="itemIsSpecial">
                            <span class="sale-price" v-html="getPriceAndCurrency('display_price_brutto_special', priceSwitcherIncludeVat)" />
                            <span class="old-price" v-html="getPriceAndCurrency('display_price_brutto', priceSwitcherIncludeVat)" />
                        </template>

                        <template v-else-if="itemHasCheapPrice">
                            <span class="minimal cheapest-label">{{ $t('cheap_price_label') }}</span>
                            <span v-html="getCheapPriceAndCurrency(priceSwitcherIncludeVat)" />
                        </template>

                        <template v-else>
                            <span v-html="getPriceAndCurrency('display_price_brutto', priceSwitcherIncludeVat)" />
                        </template>

                        <div v-if="itemOrig.final_price_item.priceinfo !== null" class="unit-price-wrp">
                            <span class="price" v-text="getPriceAndCurrency('priceinfo', false)" />
                            <span class="label" v-text="'/l'" />
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </div>
</template>

<script>
import { mapState, mapGetters, mapMutations } from 'vuex';
import _ from 'lodash';

export default {
    name: 'ProductListingCard',

    props: {
        itemOrig: {
            type: Object,
            required: true,
        },
        isSlider: {
            type: Boolean,
            default: false,
            required: false,
        },
        list: {
            type: String,
            required: false,
            default: '',
        },
    },

    data() {
        return {
            productUrl: '/#/detail',
            itemData: {},
        };
    },

    computed: {
        ...mapState({
            priceSwitcherIncludeVat: state => state.modPrices.priceSwitcherIncludeVat,
        }),
        ...mapGetters({
            getApiLocale: 'modApiResources/getApiLocale',
            productIsSpecial: 'modPrices/productIsSpecial',
            productGetTierPricesByGroupId: 'modPrices/productGetTierPricesByGroupId',
            productHasTierPricesByGroupId: 'modPrices/productHasTierPricesByGroupId',
            getTaxClassByLabel: 'modPrices/getTaxClassByLabel',
            getPriceAndCurrencyDecFmt: 'modPrices/getPriceAndCurrencyDecFmt',
        }),
        classesExcl: function () {
            return null;
        },
        routeUrlPds: function () {
            let locale = this.getApiLocale;

            // direct url
            if (this.isSlider) {
                if (locale !== 'de') {
                    return '/' + locale + '/' + _.join([this.itemOrig.url_pds], '/');
                }

                return '/' + _.join([this.itemOrig.url_pds], '/');
            }

            // with category level
            //if(this.session.categorySelected) {
            //    return '/' + _.join([this.session.categorySelected.url_path, this.itemOrig.url_pds], '/');
            //}

            if (locale !== 'de') {
                return '/' + locale + '/' + this.itemOrig.url_pds;
            }

            return '/' + this.itemOrig.url_pds;
        },
        routeUrlProductImg: function () {
            if (process.env.API_TYPE === 'sw') {
                return this.itemData.image;
            }

            // If customer domain isset get live images
            if (!_.isEmpty(process.env.CUSTOMER_DOMAIN)) {
                let image = this.itemData.image;
                return _.join([process.env.CUSTOMER_DOMAIN, 'images/catalog/thumbnails/cache/200', image], '/');
            }

            // If no customer domain isset get images from api
            let path = _.trim(process.env.config.IMG_BASE_URL, '/');
            return path + '/images/catalog/product/180x/' + this.itemData.image;
        },
        item: function () {
            return this.itemOrig;
        },
        itemIsSimple: function () {
            return this.item.type === 'simple';
        },
        itemIsGrouped: function () {
            return this.item.type === 'grouped';
        },
        itemIsConfigurable: function () {
            return this.item.type === 'configurable';
        },
        itemIsMinimal: function () {
            return false;
        },
        itemIsNew: function () {
            if (this.itemData.status) {
                let td = Date.parse(Date()),
                    startDate = Date.parse(this.itemData.status.is_new_from_date),
                    endDate = Date.parse(this.itemData.status.is_new_to_date);

                return startDate <= td && td <= endDate;
            }

            return false;
        },
        itemIsSpecial: function () {
            return this.productIsSpecial(this.itemData);
        },
        itemDiscountPercent: function () {
            let oldPrice = this.itemData.final_price_item['display_price_brutto'],
                specialPrice = this.itemData.final_price_item['display_price_brutto_special'],
                decrease = oldPrice - specialPrice,
                decreasePercentage = (decrease / oldPrice) * 100;

            return '-' + _.round(decreasePercentage) + ' %';
        },
        itemTierPrices: function () {
            let groupID = 0;

            return this.productGetTierPricesByGroupId(this.itemData, groupID);
        },
        itemTierPriceMin: function () {
            return _.minBy(this.itemTierPrices, 'display_price_brutto');
        },
        itemHasTierPrices: function () {
            let groupID = 0;

            return this.productHasTierPricesByGroupId(this.itemData, groupID);
        },
        itemTierPriceDiscount: function () {
            if (!this.itemHasTierPrices) {
                return null;
            }

            let _item_min_price = this.itemData.final_price_item.min_price;
            let _tier_min_price = this.itemTierPriceMin.price;

            let _diff = 100 - (_tier_min_price / _item_min_price) * 100;

            return _.round(_diff) + '%';
        },
        itemHasCheapPrice: function () {
            return this.itemData.cheap_price_item && this.itemData.cheap_price_item.status === 'OK';
        },
        itemTaxClass: function () {
            return this.getTaxClassByLabel(this.itemData.final_price_item.tax_class_id);
        },
    },

    created() {
        // use copy of loaded item based on vuex store
        this.itemData = this.itemOrig;

        this.$on('update-item-data', function (payload) {
            this.itemData = payload;
        });
    },

    methods: {
        ...mapMutations({
            setOpenDetail: 'modApiProduct/setOpenDetail',
            setProductId: 'modApiProduct/setProductId',
        }),
        getPriceAndCurrency: function (key, addVat) {
            return this.getPriceAndCurrencyDecFmt(this.itemData.final_price_item[key], addVat, this.itemTaxClass);
        },
        getTierPriceMinAndCurrency: function (addVat) {
            return this.getPriceAndCurrencyDecFmt(this.itemTierPriceMin.price, addVat, this.itemTaxClass);
        },
        getCheapPriceAndCurrency: function (addVat) {
            return this.getPriceAndCurrencyDecFmt(this.item.final_price_item.min_price, addVat, this.itemTaxClass);
        },
        openDetailPage: function () {
            this.gtmProductClick();

            // set necessary data for dynamic route middleware
            this.setOpenDetail(true);
            this.setProductId(this.itemData.id);

            this.$router.push({ path: this.routeUrlPds });
        },
        gtmProductClick: function () {
            if (this.$gtm) {
                let price = this.getPriceAndCurrency('display_price_brutto', this.priceSwitcherIncludeVat);

                if (this.itemIsSpecial) {
                    price = this.getPriceAndCurrency('display_price_brutto_special', this.priceSwitcherIncludeVat);
                }

                this.$gtm.pushEvent({
                    event: 'productClick',
                    ecommerce: {
                        click: {
                            actionField: {
                                list: this.list,
                            },
                            products: [
                                {
                                    name: this.itemData.name,
                                    id: this.itemData.id,
                                    price: price,
                                    brand: this.itemData.manufacturer_name,
                                },
                            ],
                        },
                        impressions: undefined,
                        detail: undefined,
                        add: undefined,
                        remove: undefined,
                        purchase: undefined,
                    },
                });
            }
        },
    },
};
</script>
