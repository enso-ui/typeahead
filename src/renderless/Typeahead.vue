<script>
import debounce from 'lodash/debounce';

export default {
    name: 'CoreTypeahead',

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
        loading: false,
        query: null,
    }),
    computed: {
        hasError() {
            return this.query && !this.regExp.test(this.query);
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
            if (this.hasError) {
                return;
            }

            if (!this.query) {
                this.items = [];
                return;
            }

            this.loading = true;

            axios.get(this.source, this.requestParams)
                .then(({ data }) => {
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

            if (items.length) {
                this.query = items[this.currentIndex][this.label];
                this.$emit('selected', items[this.currentIndex]);
                this.currentIndex = 0;
                this.query = '';
            }
        },
        nextIndex() {
            if (this.loading) {
                return;
            }

            if (++this.currentIndex > this.items.length - 1) {
                this.currentIndex = 0;
            }

            this.scrollIntoView();
        },
        previousIndex() {
            if (this.loading) {
                return;
            }

            if (--this.currentIndex < 0) {
                this.currentIndex = this.items.length - 1;
            }

            this.scrollIntoView();
        },
        updateCurrentIndex(index) {
            this.currentIndex = index;
        },
        scrollIntoView() {
            const options = this.$el.querySelectorAll('.dropdown-item.option');

            options[this.currentIndex]
                .scrollIntoView({ behavior: 'smooth', block: 'nearest' });
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
            items: this.filter(this.items),
            loading: this.loading,
            hasError: this.hasError,
            currentIndex: this.currentIndex,
            highlight: this.highlight,
            clearBindings: {
                click: this.clear,
            },
            itemEvents: index => ({
                mouseover: () => this.updateCurrentIndex(index),
                mousedown: () => this.select(),
            }),
            inputBindings: {
                value: this.query,
            },
            inputEvents: {
                input: (e) => {
                    this.query = e.target.value;
                    this.fetch();
                },
                keydown: (e) => {
                    if (e.key === 'ArrowUp') {
                        this.previousIndex();
                    }
                    if (e.key === 'ArrowDown') {
                        this.nextIndex();
                    }
                    if (e.key === 'Enter') {
                        this.select();
                    }
                },
                keyup: (e) => {
                    if (e.key === 'Escape') {
                        this.clear();
                    }
                },
            },
        });
    },
};
</script>
