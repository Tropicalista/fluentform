<template>
    <div id="print_view" class="ff_report_viewer">
        <div class="ff_nav_top">
            <div class="ff_nav_title">
                <h3>Visual Data Reporting</h3>
                <div class="ff_nav_sub_actions">
                </div>
            </div>
            <div class="ff_nav_action">
                <el-button @click="gotoRegularEntries()" type="primary" size="mini">View Regular Entries</el-button>
            </div>
        </div>
        <hr>
        <el-row v-loading="loading" :gutter="20" style="min-height: 260px;">
            <el-col class="all_report_items" :xs="24" :sm="18" :md="18" :lg="18">
                <div v-if="loading">
                    <h3>Fecthing Data.. Please wait</h3>
                </div>
                <div v-if="!loading" class="ff_report_body">

                    <div class="ff_report_card">
                        <div class="report_header">
                           Entries over last 30 days
                        </div>
                        <div class="report_body">
                            <entries-chart :form_id="form_id"></entries-chart>
                        </div>
                    </div>

                    <report-card
                        v-for="(report,report_key) in report_items"
                        :key="report_key"
                        :form_id="form_id"
                        :report_key="report_key"
                        :report_indexes="reportIndexes"
                        :report="report"></report-card>
                </div>
                <div v-if="(!total_entries && !loading) || (!reportIndexes.length && !loading)"
                     class="no_entries_found">
                    <p>Sorry! No reports found based on your filter, available form submissions and input
                        types</p>
                </div>
            </el-col>
            <el-col class="ff_print_hide" :xs="24" :sm="6" :md="6" :lg="6">
                <div class="entry_info_box postbox">
                    <div class="entry_info_header">
                        <b>Filter Data by Status</b>
                    </div>
                    <div class="entry_info_body report_status_filter">
                        <el-checkbox-group @change="fetchReport()" v-model="filter_statuses">
                            <el-checkbox v-for="(status, status_key) in entry_statuses" :key="status_key"
                                         :label="status_key">{{status}}
                            </el-checkbox>
                        </el-checkbox-group>
                        <p style="margin-top: 10px" v-show="!filter_statuses.length">Show from all except trashed</p>
                    </div>
                </div>
                <div class="entry_info_box postbox">
                    <div class="entry_info_header">
                        <b>Other Info</b>
                    </div>
                    <div class="entry_info_body">
                        <p>Total Entries: {{total_entries}}</p>
                        <hr/>
                        <p><b>Entries By Browser</b></p>
                        <ul>
                            <li v-for="(browserCount,browsername) in browsers">{{browsername}}: {{browserCount}}</li>
                        </ul>
                        <hr/>
                        <p><b>Entries By Device</b></p>
                        <ul>
                            <li v-for="(deviceCount,deviceName) in devices">{{deviceName}}: {{deviceCount}}</li>
                        </ul>
                    </div>
                </div>
                <el-button @click="printReport()" size="mini" type="info">Print this report</el-button>
                <el-button @click="resetAnalytics()" size="mini" type="info">Reset Form Analytics</el-button>
            </el-col>
        </el-row>
    </div>
</template>

<script type="text/babel">
    import ReportCard from './reportCard';
    import each from 'lodash/each';
    import EntriesChart from '../../AllEntries/Components/chartView';

    export default {
        name: 'visual_report',
        props: ['form_id'],
        components: {
            EntriesChart,
            ReportCard
        },
        data() {
            return {
                loading: true,
                total_entries: 0,
                entry_statuses: window.fluent_form_entries_vars.entry_statuses,
                filter_statuses: [],
                report_items: {},
                browsers: {},
                devices: {},
            }
        },
        computed: {
            reportIndexes() {
                let items = [];
                each(this.report_items, (item, item_name) => {
                    items.push(item_name);
                });
                return items;
            }
        },
        methods: {
            fetchReport() {
                this.loading = true;
                jQuery.get(window.ajaxurl, {
                    form_id: this.form_id,
                    action: 'fluentform-form-report',
                    statuses: this.filter_statuses
                })
                    .then(reponse => {
                        this.report_items = reponse.data.report_items;
                        this.total_entries = parseInt(reponse.data.total_entries);
                        this.browsers = reponse.data.browsers;
                        this.devices = reponse.data.devices;
                    })
                    .always(() => {
                        this.loading = false;
                    });
            },
            resetAnalytics() {
                if (window.confirm('All the views recorded to this form will be deleted. Are you sure?')) {
                    jQuery.post(window.ajaxurl, {
                        form_id: this.form_id,
                        action: 'fluentform-reset-analytics'
                    })
                        .then(response => {
                            this.$message.success(response.data.message);
                        });
                }
            },
            gotoRegularEntries() {
                this.$router.push({
                    name: 'form-entries'
                });
            },
            printReport() {
                window.print();
            }
        },
        mounted() {
            each(this.entry_statuses, (item, key) => {
                if (key != 'trashed') {
                    this.filter_statuses.push(key);
                }
            });
            this.fetchReport();
        }
    }
</script>

