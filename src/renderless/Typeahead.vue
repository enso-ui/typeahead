<script>
import debounce from 'lodash/debounce';

export default {
    name: 'CoreTypeahead',

    model: {
        event: 'selected',
    },

    props: {
        disabled: {
            type: Boolean,
            default: false,
        },
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
        i18n: {
            type: Function,
            default: v => v,
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
        clear() {
            this.query = null;
            this.items = [];
        },
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
        highlight(item) {
            this.query.split(' ').filter(word => word.length)
                .forEach(word => (item = item.replace(
                    new RegExp(`(${word})`, 'gi'), '<b>$1</b>',
                )));

            return item;
        },
        isCurrent(index) {
            return index === this.currentIndex;
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
        updateCurrent(index) {
            this.currentIndex = index;
        },
    },
    render() {
        return this.$scopedSlots.default({
            clearBindings: { click: this.clear },
            disabled: this.disabled,
            hasError: this.hasError,
            highlight: this.highlight,
            i18n: this.i18n,
            inputBindings: { value: this.query },
            inputEvents: {
                input: (e) => {
                    this.query = e.target.value;
                    this.fetch();
                },
                keyup: (e) => {
                    if (e.key === 'Escape') {
                        this.clear();
                    }
                },
            },
            itemEvents: index => ({
                mouseover: () => this.updateCurrentIndex(index),
                mousedown: () => this.select(),
            }),
            isCurrent: this.isCurrent,
            items: this.filter(this.items),
            label: this.label,
            loading: this.loading,
            query: this.query,
            select: this.select,
            updateCurrent: this.updateCurrent,
        });
    },
};
</script>
