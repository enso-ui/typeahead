<script>
import debounce from 'lodash/debounce';

export default {
    model: {
        event: 'selected',
    },
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
        value: {
            type: [String, Object],
            default: null,
        },
    },
    data: () => ({
        currentIndex: 0,
        items: [],
        hiddenDropdown: true,
        loading: false,
        query: null,
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
        clear() {
            this.query = null;
            this.items = [];
        },
        select() {
            const items = this.filter(this.items);
            if (this.visibleDropdown && items.length) {
                this.query = items[this.currentIndex][this.label];
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
            query: this.query,
            label: this.label,
            loading: this.loading,
            items: this.filter(this.items),
            hasError: this.hasError,
            visibleDropdown: this.visibleDropdown,
            currentIndex: this.currentIndex,
            clear: this.clear,
            updateIndex: this.updateIndex,
            highlight: this.highlight,
            inputBindings: {
                value: this.query,
            },
            itemEvents: {
                mousedown: () => this.select(),
            },
            inputEvents: {
                input: (e) => {
                    this.query = e.target.value;
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
                        this.select();
                    }
                    if (e.key === 'Escape') {
                        this.clear();
                    }
                },
            },
        });
    },
};
</script>
