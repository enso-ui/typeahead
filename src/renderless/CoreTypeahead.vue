<script>
import debounce from 'lodash/debounce';
import Modes from '@enso-ui/search-mode/src/modes';

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
        searchMode: {
            type: String,
            default: 'full',
            validator: v => Modes.includes(v),
        },
        searchModes: {
            type: Array,
            default: () => ['full'],
            validator: v => v.every(mode => Modes.includes(mode)),
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

    data: v => ({
        items: [],
        loading: false,
        mode: v.searchMode,
        ongoingRequest: null,
        query: '',
    }),

    computed: {
        hasError() {
            return this.query && !this.regExp.test(this.query);
        },
        modeSelector() {
            return this.searchModes.length > 1;
        },
        requestParams() {
            return {
                params: {
                    query: this.query,
                    paginate: this.paginate,
                    params: this.params,
                    searchMode: this.mode,
                },
                cancelToken: this.ongoingRequest.token,
            };
        },
    },

    created() {
        this.fetch = debounce(this.fetch, this.debounce);
    },

    methods: {
        clear() {
            this.query = '';
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

            if (this.ongoingRequest) {
                this.ongoingRequest.cancel();
            }

            this.ongoingRequest = axios.CancelToken.source();
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
        select(index) {
            const items = this.filter(this.items);

            if (items.length) {
                this.query = items[index][this.label];
                this.$emit('selected', items[index]);
            }
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
                select: () => this.select(index),
            }),
            items: this.filter(this.items),
            label: this.label,
            loading: this.loading,
            modeBindings: {
                modes: this.searchModes,
                query: this.query,
                value: this.mode,
            },
            modeEvents: {
                input: event => (this.mode = event),
                change: this.fetch,
            },
            modeSelector: this.modeSelector,
            query: this.query,
            select: this.select,
        });
    },
};
</script>
