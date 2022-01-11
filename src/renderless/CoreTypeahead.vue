<script>
import { debounce } from 'lodash';
import Modes from '@enso-ui/search-mode/src/modes';

export default {
    name: 'CoreTypeahead',

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
            default: error => {
                throw error;
            },
        },
        filter: {
            type: Function,
            default: items => (items),
        },
        forceSelection: {
            type: Boolean,
            default: false,
        },
        http: {
            required: true,
            type: Function,
        },
        i18n: {
            type: Function,
            default: v => v,
        },
        label: {
            type: String,
            default: 'label',
        },
        minQueryLength: {
            type: Number,
            default: 3,
        },
        paginate: {
            type: Number,
            default: 100,
        },
        params: {
            type: Object,
            default: null,
        },
        readonly: {
            type: Boolean,
            default: false,
        },
        regExp: {
            type: RegExp,
            default() {
                return /(.*?)/;
            },
        },
        searchControl: {
            type: Boolean,
            default: false,
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
        taggable: {
            type: Boolean,
            default: false,
        },
    },

    emits: ['add-tag', 'search', 'selected'],

    data: v => ({
        items: [],
        loading: false,
        mode: v.searchMode,
        ongoingRequest: null,
        query: '',
        waitingResponse: false,
    }),

    computed: {
        canAddTag() {
            return this.taggable && !!this.query
                && this.queryDoesntMatch && !this.waitingResponse;
        },
        invalidQuery() {
            return this.query && !this.regExp.test(this.query);
        },
        modeSelector() {
            return this.searchModes.length > 1;
        },
        queryDoesntMatch() {
            return !this.items
                .some(item => `${item[this.label]}`
                    .toLowerCase() === this.query.toLowerCase());
        },
        requestParams() {
            return {
                query: this.query,
                paginate: this.paginate,
                params: this.params,
                searchMode: this.mode,
            };
        },
    },

    watch: {
        query(query) {
            this.waitingResponse = query !== '';
        },
    },

    created() {
        this.fetch = debounce(this.fetch, this.debounce);
    },

    methods: {
        addTag() {
            this.$emit('add-tag', this.query);
        },
        bold(label, arg) {
            let from;

            try {
                from = new RegExp(`(${arg})`, 'gi');
            } catch {
                from = arg;
            }

            return `${label}`.replace(from, '<b>$1</b>');
        },
        clear() {
            this.query = '';
            this.items = [];
        },
        fetch() {
            if (this.invalidQuery) {
                return;
            }

            if (this.query.length < this.minQueryLength) {
                this.items = [];
                return;
            }

            if (this.ongoingRequest) {
                this.ongoingRequest.cancel();
            }

            this.ongoingRequest = this.http.CancelToken.source();
            this.loading = true;

            this.http.get(this.source, {
                params: this.requestParams,
                cancelToken: this.ongoingRequest.token,
            }).then(({ data }) => {
                this.items = data;
                this.loading = false;
                this.waitingResponse = false;
            }).catch(error => {
                this.loading = false;
                this.waitingResponse = false;
                this.errorHandler(error);
            });
        },
        highlight(item) {
            this.query.split(' ')
                .filter(arg => arg.length)
                .forEach(arg => (item = this.bold(item, arg)));

            return item;
        },
        search(item = null) {
            if (this.query.length < this.minQueryLength) {
                return;
            }

            if (this.forceSelection && !item && this.items.length > 0) {
                [item] = this.items;
            }

            this.$emit('search', { item, query: this.query });
            this.query = '';
        },
        select(index) {
            const items = this.filter(this.items);

            if (items.length) {
                this.$emit('selected', items[index]);
                this.search(items[index]);
            }
        },
    },
    render() {
        return this.$slots.default({
            addTag: {
                click: this.addTag,
            },
            canAddTag: this.canAddTag,
            clearBindings: { click: this.clear },
            controlEvents: {
                click: this.search,
            },
            disabled: this.disabled,
            invalidQuery: this.invalidQuery,
            highlight: this.highlight,
            i18n: this.i18n,
            inputBindings: { value: this.query },
            inputEvents: selection => ({
                input: e => {
                    this.query = e.target.value;
                    this.fetch();
                },
                keydown: e => {
                    switch (e.key) {
                    case 'Escape':
                        this.clear();
                        break;
                    case 'Enter':
                        if (this.canAddTag && e.shiftKey) {
                            this.addTag();
                        } else if (!selection) {
                            this.search();
                        }
                        break;
                    default:
                        break;
                    }
                },
            }),
            itemEvents: index => ({
                select: () => this.select(index),
            }),
            items: this.filter(this.items),
            label: this.label,
            loading: this.loading,
            minQueryLength: this.minQueryLength,
            modeBindings: {
                modes: this.searchModes,
                query: this.query,
                modelValue: this.mode,
            },
            modeEvents: {
                'update:modelValue': event => (this.mode = event),
                change: this.fetch,
            },
            modeSelector: this.modeSelector,
            readonly: this.readonly,
            query: this.query,
            searchControl: this.searchControl,
        });
    },
};
</script>
