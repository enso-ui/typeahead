<script>

import debounce from 'lodash/debounce';

export default {
    props: {
        debounce: {
            type: Number,
            default: 250,
            validator: v => v > 0,
        },
        errorHandler: {
            type: Function,
            default: (error) => {
                throw error;
            },
        },
        filter: {
            type: Function,
            default: items => (items),
        },
        label: {
            type: String,
            default: 'label',
        },
        paginate: {
            type: Number,
            default: 100,
        },
        params: {
            type: Object,
            default: null,
        },
        query: {
            type: String,
            default: '',
        },
        regExp: {
            type: RegExp,
            default() {
                return /(.*?)/;
            },
        },
        source: {
            type: String,
            required: true,
        },
    },

    data: () => ({
        currentIndex: 0,
        items: [],
        hiddenDropdown: true,
        loading: false,
    }),

    computed: {
        hasError() {
            return this.query && !this.regExp.test(this.query);
        },
        visibleDropdown() {
            return !this.hiddenDropdown && !!this.query && !this.hasError;
        },
        requestParams() {
            return {
                params: {
                    query: this.query,
                    paginate: this.paginate,
                    params: this.params,
                },
            };
        },
    },

    created() {
        this.fetch = debounce(this.fetch, this.debounce);
    },

    methods: {
        fetch() {
            if (!this.query || this.hasError) {
                return;
            }

            this.loading = true;

            axios.get(this.source, this.requestParams)
                .then(({ data }) => {
                    this.hiddenDropdown = false;
                    this.items = data;
                    this.loading = false;
                }).catch((error) => {
                    this.loading = false;
                    this.errorHandler(error);
                });
        },
        update(query = '') {
            if (!query) {
                this.items = [];
            }

            this.$emit('input', query);
        },
        hit() {
            const items = this.filter(this.items);

            if (this.visibleDropdown && items.length) {
                this.update(items[this.currentIndex][this.label]);
                this.$emit('update', items[this.currentIndex]);
                this.$emit('selected', items[this.currentIndex]);
                this.hiddenDropdown = true;
            }
        },
        keyUp() {
            if (this.currentIndex > 0) {
                this.currentIndex--;
            }
        },
        keyDown() {
            if (this.currentIndex < this.items.length - 1) {
                this.currentIndex++;
            }
        },
        updateIndex(index) {
            this.currentIndex = index;
        },
        highlight(item) {
            this.query.split(' ')
                .filter(word => word.length)
                .forEach((word) => {
                    item = item.replace(
                        new RegExp(`(${word})`, 'gi'), '<b>$1</b>',
                    );
                });

            return item;
        },
    },

    render() {
        return this.$scopedSlots.default({
            loading: this.loading,
            items: this.filter(this.items),
            hasError: this.hasError,
            visibleDropdown: this.visibleDropdown,
            currentIndex: this.currentIndex,
            updateIndex: this.updateIndex,
            highlight: this.highlight,
            itemEvents: {
                mousedown: () => this.hit(),
            },
            inputEvents: {
                input: (e) => {
                    this.update(e.target.value);
                    this.fetch();
                },
                keydown: (e) => {
                    if (e.key === 'ArrowUp') {
                        this.keyUp();
                    }

                    if (e.key === 'ArrowDown') {
                        this.keyDown();
                    }

                    if (e.key === 'Enter') {
                        this.hit();
                    }

                    if (e.key === 'Escape') {
                        this.update();
                    }
                },
            },
        });
    },
};

</script>
