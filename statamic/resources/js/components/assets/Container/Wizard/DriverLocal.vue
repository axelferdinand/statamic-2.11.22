<template>

    <div class="form-group">
        <label class="block">{{ translate('cp.path') }}</label>
        <small class="help-block">{{ translate('cp.asset_path_instructions') }}</small>
        <div class="input-with-loader">
            <input type="text" class="form-control" v-model="path" @keyup="resolvePath | debounce 500" />
            <span v-show="resolvingPath" class="icon-resolving icon icon-circular-graph animation-spin"></span>
        </div>
        <small class="help-block" v-if="showResolvedPath">
            <span>
                {{ translate('cp.path_resolves_to', { path: resolvedPath }) }}
                <span v-show="resolvedPathExists" class="text-success">{{ translate('cp.path_exists') }}</span>
                <span v-else class="text-danger">{{ translate('cp.path_does_not_exist') }}</span>
            </span>
        </small>
    </div>

    <div class="form-group" v-if="resolvedPathExists">
        <label class="block">{{ translate('cp.url') }}</label>
        <small class="help-block">{{ translate('cp.asset_url_instructions') }}</small>
        <div class="input-with-loader">
            <input type="text" class="form-control" v-model="url" @keyup="resolveUrl | debounce 500" />
            <span v-show="resolvingUrl" class="icon-resolving icon icon-circular-graph animation-spin"></span>
        </div>
        <small class="help-block" v-if="showResolvedUrl">
            {{ translate('cp.url_resolves_to', { path: resolvedUrl }) }}
            <span class="text-success" v-show="validUrl">{{ translate('cp.valid_url') }}.</span>
            <span class="text-danger" v-else>{{ translate('cp.invalid_url') }}</span>
        </small>
    </div>

    <div class="form-group" v-if="!editing">
        <button class="btn btn-default" @click="submit" :disabled="!canContinue">{{ translate('cp.next_step') }}</button>
    </div>

</template>

<style lang="scss">
    .input-with-loader {
        position: relative;

        .icon-resolving {
            position: absolute;
            top: 7px;
            right: 5px;
        }
    }
</style>

<script>
    export default {

        props: {
            path: String,
            url: String,
            editing: {
                type: Boolean,
                default() {
                    return false;
                }
            }
        },


        data() {
            return {
                resolvedPath: null,
                resolvingPath: false,
                resolvedPathExists: false,
                resolvedUrl: null,
                resolvingUrl: false
            }
        },


        computed: {

            showResolvedPath() {
                return this.resolvedPath || this.resolvingPath;
            },

            showResolvedUrl() {
                return this.resolvedUrl || this.resolvingUrl;
            },

            validUrl() {
                return this.showResolvedUrl.substr(0, 4) === 'http';
            },

            canContinue() {
                return this.resolvedPathExists;
            }

        },


        ready() {
            if (this.editing) {
                // For whatever reason, it doesn't work unless there's a timeout. ??\_(???)_/??
                setTimeout(() => this.resolvePath(), 0);
            }
        },


        watch: {

            resolvedPathExists(exists) {
                if (exists) {
                    this.resolveUrl();
                }
            }

        },


        methods: {

            resolvePath() {
                this.resolvingPath = true;

                this.$http.post(cp_url('assets/containers/resolve-path'), {
                    path: this.path
                }, function (response) {
                    this.resolvingPath = false;
                    this.resolvedPath = response.path;
                    this.resolvedPathExists = response.exists;
                });
            },

            resolveUrl() {
                this.resolvingUrl = true;

                this.$http.post(cp_url('assets/containers/resolve-url'), {
                    url: this.url
                }, function (response) {
                    this.resolvingUrl = false;
                    this.resolvedUrl = response.url;
                });
            },

            submit() {
                this.$emit('submit');
            }

        }

    }
</script>
