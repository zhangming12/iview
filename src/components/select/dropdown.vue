<template>
    <div class="ivu-select-dropdown" :class="className" :style="styles"><slot></slot></div>
</template>
<script>
    import Vue from 'vue';
    const isServer = Vue.prototype.$isServer;
    import { getStyle } from '../../utils/assist';
    const Popper = isServer ? function() {} : require('popper.js/dist/umd/popper.js');  // eslint-disable-line

    export default {
        name: 'Drop',
        props: {
            placement: {
                type: String,
                default: 'bottom-start'
            },
            className: {
                type: String
            }
        },
        data () {
            return {
                popper: null,
                width: ''
            };
        },
        computed: {
            styles () {
                let style = {};
                if (this.width) style.width = `${this.width}px`;
                return style;
            }
        },
        methods: {
            update () {
                if (isServer) return;
                if (this.popper) {
                    this.$nextTick(() => {
                        this.popper.update();
                    });
                } else {
                    this.$nextTick(() => {
                        this.popper = new Popper(this.$parent.$refs.reference, this.$el, {
                            placement: this.placement,
                            modifiers: {
                                computeStyle:{
                                    gpuAcceleration: false,
                                }
                            }
                        });
                    });
                }
                // set a height for parent is Modal and Select's width is 100%
                if (this.$parent.$options.name === 'iSelect') {
                    this.width = parseInt(getStyle(this.$parent.$el, 'width'));
                }
            },
            destroy () {
                if (this.popper) {
                    setTimeout(() => {
                        if (this.popper) {
                            this.popper.destroy();
                            this.popper = null;
                        }
                    }, 300);
                }
            }
        },
        created () {
            this.$on('on-update-popper', this.update);
            this.$on('on-destroy-popper', this.destroy);
        },
        beforeDestroy () {
            if (this.popper) {
                this.popper.destroy();
            }
        }
    };
</script>
