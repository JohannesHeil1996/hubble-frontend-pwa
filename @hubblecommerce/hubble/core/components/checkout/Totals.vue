<template>
    <div>
        <div class="summary-row sub-totals bg-light">
            <div v-text="$t('Subtotal')" />

            <div>
                <span class="float-right" v-text="getSubTotal()" />
            </div>
        </div>

        <div class="summary-row shipping-costs bg-light">
            <div v-text="$t('Shipping Costs')" />

            <div>
                <span class="float-right" v-text="getShippingCosts()" />
            </div>
        </div>

        <div class="summary-row totals bg-light">
            <div v-text="$t('Totals')" />

            <div>
                <span class="float-right" v-text="getTotal()" />
            </div>
        </div>
    </div>
</template>

<script>
import { mapGetters } from 'vuex';

export default {
    name: 'Totals',

    computed: {
        ...mapGetters({
            getTotals: 'modCart/getTotals',
            getSubtotals: 'modCart/getSubtotals',
            priceDecFmt: 'modPrices/priceDecFmt',
            priceAddCur: 'modPrices/priceAddCur',
            getShippingCostsFromStore: 'modCart/getShippingCosts',
        }),
    },

    methods: {
        getSubTotal: function () {
            let subtotals = this.getSubtotals;

            // Format subtotals
            subtotals = this.priceDecFmt(subtotals);
            subtotals = this.priceAddCur(subtotals);

            return subtotals;
        },
        getShippingCosts: function () {
            let shippingCosts = this.getShippingCostsFromStore;

            // Format shippingCosts
            shippingCosts = this.priceDecFmt(shippingCosts);
            shippingCosts = this.priceAddCur(shippingCosts);

            return shippingCosts;
        },
        getTotal: function () {
            let total = this.getTotals;

            // Format totals
            total = this.priceDecFmt(total);
            total = this.priceAddCur(total);

            return total;
        },
    },
};
</script>
