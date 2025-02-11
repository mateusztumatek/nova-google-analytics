<template>
    <loading-view :loading='initialLoading'>
        <heading class='mb-6'>
            {{ title }}
        </heading>
        <div class='flex'>
            <div class='relative h-9 flex-no-shrink mb-6'>
                <icon class='absolute search-icon-center ml-3 text-70' type='search' />
                <input
                    v-model='search'
                    :placeholder="__('Search')"
                    class='appearance-none form-search w-search pl-search shadow'
                    data-testid='search-input'
                    dusk='search'
                    spellcheck='false'
                    type='search'
                    @search='performSearch'
                    @keydown.stop='performSearch'
                />
            </div>
        </div>
        <loading-card :loading='loading' class='card relative'>
            <div>
                <div class='flex items-center py-3 border-b border-50'>
                    <div class='flex items-center ml-auto px-3'>
                        <filter-menu
                            :perPage='limit'
                            :perPageOptions='[10, 25, 50, 100]'
                            :viaResource='false'
                            @per-page-changed='updateLimit'
                        ></filter-menu>
                    </div>
                </div>
                <table
                    v-if='data.length > 0'
                    cellpadding='0'
                    cellspacing='0'
                    class='table w-full table-default'
                >
                    <thead>
                        <tr>
                            <th class='text-left'>
                                <sortable-icon
                                    :direction='direction'
                                    resource-name='Pages'
                                    uri-key='ga:pageTitle'
                                    @reset='resetOrderBy'
                                    @sort='sortByChange'
                                >
                                    <span class='inline-flex items-center'>
                                        {{ __('Name') }}
                                    </span>
                                </sortable-icon>
                            </th>

                            <th class='text-left'>
                                <sortable-icon
                                    :direction='direction'
                                    resource-name='Paths'
                                    uri-key='ga:pagePath'
                                    @reset='resetOrderBy'
                                    @sort='sortByChange'
                                >
                                    <span class='inline-flex items-center'>
                                        {{ __('Path') }}
                                    </span>
                                </sortable-icon>
                            </th>

                            <th class='text-left'>
                                <sortable-icon
                                    :direction='direction'
                                    resource-name='Pages'
                                    uri-key='ga:pageviews'
                                    @reset='resetOrderBy'
                                    @sort='sortByChange'
                                >
                                    <span class='inline-flex items-center'>
                                        {{ __('Visits') }}
                                    </span>
                                </sortable-icon>
                            </th>

                            <th class='text-left'>
                                <sortable-icon
                                    :direction='direction'
                                    resource-name='Pages'
                                    uri-key='ga:uniquePageviews'
                                    @reset='resetOrderBy'
                                    @sort='sortByChange'
                                >
                                    <span class='inline-flex items-center'>
                                        {{ __('Unique Visits') }}
                                    </span>
                                </sortable-icon>
                            </th>

                            <th class='text-left'>
                                <sortable-icon
                                    :direction='direction'
                                    resource-name='Pages'
                                    uri-key='ga:avgTimeOnPage'
                                    @reset='resetOrderBy'
                                    @sort='sortByChange'
                                >
                                    <span class='inline-flex items-center'>
                                        {{ __('Avg. Time on Page') }}
                                    </span>
                                </sortable-icon>
                            </th>

                            <th class='text-left'>
                                <sortable-icon
                                    :direction='direction'
                                    resource-name='Pages'
                                    uri-key='ga:entrances'
                                    @reset='resetOrderBy'
                                    @sort='sortByChange'
                                >
                                    <span class='inline-flex items-center'>
                                        {{ __('Entrances') }}
                                    </span>
                                </sortable-icon>
                            </th>

                            <th class='text-left'>
                                <sortable-icon
                                    :direction='direction'
                                    resource-name='Pages'
                                    uri-key='ga:bounceRate'
                                    @reset='resetOrderBy'
                                    @sort='sortByChange'
                                >
                                    <span class='inline-flex items-center'>
                                        {{ __('Bounce Rate') }}
                                    </span>
                                </sortable-icon>
                            </th>

                            <th class='text-left'>
                                <sortable-icon
                                    :direction='direction'
                                    resource-name='Pages'
                                    uri-key='ga:exitRate'
                                    @reset='resetOrderBy'
                                    @sort='sortByChange'
                                >
                                    <span class='inline-flex items-center'>
                                        {{ __('Exit Rate') }}
                                    </span>
                                </sortable-icon>
                            </th>

                            <th class='text-left'>
                                <sortable-icon
                                    :direction='direction'
                                    resource-name='Pages'
                                    uri-key='ga:pageValue'
                                    @reset='resetOrderBy'
                                    @sort='sortByChange'
                                >
                                    <span class='inline-flex items-center'>
                                        {{ __('Page Value') }}
                                    </span>
                                </sortable-icon>
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for='row in data'>
                            <td>{{ row.name }}</td>
                            <td>{{ row.path }}</td>
                            <td>{{ row.visits }}</td>
                            <td>{{ row.unique_visits }}</td>
                            <td>{{ getFormattedTime(row.avg_page_time) }}</td>
                            <td>{{ row.entrances }}</td>
                            <td>{{ getFormattedPercent(row.bounce_rate) }}</td>
                            <td>{{ getFormattedPercent(row.exit_rate) }}</td>
                            <td>{{ getFormattedCurrency(row.page_value) }}</td>
                        </tr>
                    </tbody>
                </table>
            </div>

            <pagination-links
                :current-page='page'
                :data='data'
                :hasMore='hasMore'
                :hasPrevious='hasPrevious'
                :total-pages='totalPages'
                @next='nextPage'
                @previous='previousPage'
            ></pagination-links>
        </loading-card>
    </loading-view>
</template>

<script>
import PaginationLinks from './PaginationLinks.vue';
import dayjs from 'dayjs';
import duration from 'dayjs/plugin/duration';
import utc from 'dayjs/plugin/utc';
import FilterMenu from './FilterMenu';

dayjs.extend(duration);
dayjs.extend(utc);

export default {
    components: {
        'pagination-links': PaginationLinks,
        'dayjs': dayjs,
        'filter-menu': FilterMenu,
    },
    data: function() {
        return {
            title: 'Google Analytics',
            data: [],
            duration: 'week',
            initialLoading: true,
            loading: true,
            hasMore: true,
            page: 1,
            totalPages: 1,
            search: '',
            sortBy: 'ga:pageviews',
            sortDirection: 'desc',
            limit: 10,
        };
    },
    metaInfo() {
        return {
            title: this.title,
        };
    },
    methods: {
        updateDuration(event) {
            this.duration = event.target.value;
            this.getData();
        },

        updateLimit(value) {
            this.limit = value;
            this.getData();
        },

        getData() {
            Nova.request()
                .get(`/nova-vendor/nova-google-analytics/pages?limit=${this.limit}&duration=${this.duration}&page=${this.page}&s=${this.search}&sortBy=${this.sortBy}&sortDirection=${this.sortDirection}`)
                .then(response => {
                    this.data = response.data.pageData;
                    this.totalPages = response.data.totalPages;
                    this.hasMore = response.data.hasMore;
                    this.loading = false;
                });
        },

        nextPage() {
            this.loading = true;
            this.page++;
            this.getData();
        },

        previousPage() {
            this.loading = true;
            if (this.hasPrevious) {
                this.page--;
            }
            this.getData();
        },

        performSearch(event) {
            if (event.which != 9) {
                this.page = 1;
                this.getData();
            }
        },

        sortByChange(event) {
            let direction = this.sortDirection == 'asc' ? 'desc' : 'asc';

            if (this.sortBy != event.key) {
                direction = 'asc';
            }

            this.sortBy = event.key;
            this.sortDirection = direction;
            this.getData();
        },

        resetOrderBy(event) {
            this.sortBy = 'ga:pageviews';
            this.sortDirection = 'desc';
            this.getData();
        },

        getFormattedTime(timeString) {
            return dayjs.utc(dayjs.duration({ seconds: timeString }).asMilliseconds()).format('HH:mm:ss');
        },

        getFormattedPercent(percentString) {
            return parseFloat(percentString).toFixed(2) + '%';
        },

        getFormattedCurrency(percentString) {
            return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(percentString);
        },
    },
    computed: {
        hasPrevious() {
            return this.page > 1;
        },
    },
    mounted() {
        this.getData();
        this.initialLoading = false;
    },
};
</script>
