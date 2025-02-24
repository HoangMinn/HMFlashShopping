<template>
    <div class="max-width-1200">
        <div class="flexbox-annotated-section">
            <div class="flexbox-annotated-section-annotation">
                <div class="annotated-section-title pd-all-20">
                    <h2>License</h2>
                </div>
            </div>

            <div class="flexbox-annotated-section-content">
                <div class="wrapper-content pd-all-20">
                    <div style="margin: auto; width:30px;" v-if="isLoading">
                        <half-circle-spinner
                            :animation-duration="1000"
                            :size="15"
                            color="#808080"
                        />
                    </div>
                    <div v-if="!isLoading && !verified">
                        <div class="form-group">
                            <label class="text-title-field" for="buyer">Your username on Envato</label>
                            <input type="text" class="next-input" v-model="buyer" id="buyer" placeholder="Your Envato's username">
                            <div>
                                <small>If your profile page is <a href="https://codecanyon.net/user/john-smith" rel="nofollow">https://codecanyon.net/user/john-smith</a>, then your username on Envato is <strong>john-smith</strong>.</small>
                            </div>
                        </div>
                        <div class="form-group">
                            <div>
                                <div class="float-left">
                                    <label class="text-title-field" for="purchase_code">Purchase code</label>
                                </div>
                                <div class="float-right text-right">
                                    <small><a href="https://help.market.envato.com/hc/en-us/articles/202822600-Where-Is-My-Purchase-Code" target="_blank">What's this?</a></small>
                                </div>
                                <div class="clearfix"></div>
                            </div>
                            <input type="text" class="next-input" v-model="purchaseCode" id="purchase_code" placeholder="Ex: 10101010-10aa-0101-a1b1010a01b10">
                        </div>
                        <div class="form-group">
                            <label><input type="checkbox" name="license_rules_agreement" value="1" v-model="licenseRulesAgreement">Confirm that, according to the Envato License Terms, each license entitles one person for a single project. Creating multiple unregistered installations is a copyright violation.
                                <a href="https://codecanyon.net/licenses/standard" target="_blank" rel="nofollow">More info</a>.</label>
                        </div>
                        <div class="form-group">
                            <button :class="activating ? 'btn btn-info button-loading' : 'btn btn-info'" type="button" @click="activateLicense()">Activate license</button>
                        </div>
                        <hr>
                        <div class="form-group">
                            <p><small class="text-danger">Note: Your site IP will be added to blacklist after 5 failed attempts.</small></p>
                            <p>
                                <small>A purchase code (license) is only valid for One Domain. Are you using this theme on a new domain? Purchase a
                                <a href="https://codecanyon.net/user/botble/portfolio" target="_blank" rel="nofollow">new license here</a> to get a new purchase code.</small>
                            </p>
                        </div>
                    </div>
                    <div v-if="!isLoading && verified">
                        <p class="text-info">Licensed to {{ license.licensed_to }}. Activated since {{ license.activated_at }}.</p>
                        <div class="form-group">
                            <button :class="deactivating ? 'btn btn-warning button-loading' : 'btn btn-warning'" type="button" @click="deactivateLicense()">Deactivate license</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import {HalfCircleSpinner} from 'epic-spinners'

    export default {
        components: {
            HalfCircleSpinner
        },

        props: {
            verifyUrl: {
                type: String,
                default: () => null,
                required: true
            },
            activateLicenseUrl: {
                type: String,
                default: () => null,
                required: true
            },
            deactivateLicenseUrl: {
                type: String,
                default: () => null,
                required: true
            },
        },

        data() {
            return {
                isLoading: true,
                verified: false,
                purchaseCode: null,
                buyer: null,
                licenseRulesAgreement: 0,
                activating: false,
                deactivating: false,
                license: null,
            };
        },
        mounted() {
            this.verifyLicense();
        },

        methods: {
            verifyLicense() {
                axios.get(this.verifyUrl)
                    .then(res =>  {
                        if (!res.data.error) {
                            this.verified = true;
                            this.license = res.data.data;
                        }
                        this.isLoading = false;
                    })
                    .catch(res =>  {
                        Botble.handleError(res.response.data);
                        this.isLoading = false;
                    });
            },

            activateLicense() {
                this.activating = true;
                axios.post(this.activateLicenseUrl, {purchase_code: this.purchaseCode, buyer: this.buyer, license_rules_agreement: this.licenseRulesAgreement})
                    .then(res =>  {
                        if (res.data.error) {
                            Botble.showError(res.data.message);
                        } else {
                            this.verified = true;
                            this.license = res.data.data;
                        }
                        this.activating = false;
                    })
                    .catch(res =>  {
                        Botble.handleError(res.response.data);
                        this.activating = false;
                    });
            },
            deactivateLicense() {
                this.deactivating = true;
                axios.post(this.deactivateLicenseUrl)
                    .then(res =>  {
                        if (res.data.error) {
                            Botble.showError(res.data.message);
                        } else {
                            this.verified = false;
                        }
                        this.deactivating = false;
                    })
                    .catch(res =>  {
                        Botble.handleError(res.response.data);
                        this.deactivating = false;
                    });
            },
        }
    }
</script>
