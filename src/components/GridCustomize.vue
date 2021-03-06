/*
Copyright 2019 Tom Peters

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

// TODO - handle max lengths and notification

<template>
    <section class="grid-customize">
        <form @submit.prevent="submit">
            <template v-if="errors">
                <template v-for="(errList, errKey) in errors">
                    <div class="errors" :key="errKey">
                        <h2>Error</h2>

                        <p>Please correct the following errors:</p>

                        <ul>
                            <li>{{errKey}}
                                <ul>
                                    <li v-for="err in errList" :key="err">{{err}}</li>
                                </ul>
                            </li>
                        </ul>
                    </div>
                </template>
            </template>
            <fieldset>
                <legend>General Settings</legend>

                <div class="field">
                    <label for="label" class="optional">Label</label>
                    <input type="text" id="label" name="label" v-model="form.label">
                </div>

                <div class="field">
                    <label for="event-date" class="optional">Event Date</label>
                    <input type="date" id="event-date" name="event-date" v-model="form.eventDate">
                </div>

                <div class="field radio" v-if="pool.gridType === 'roll100'">
                    <label>Rollover?</label>
                    <label><input type="radio" v-model="form.rollover" :value="false"> No</label>
                    <label><input type="radio" v-model="form.rollover" :value="true"> Yes</label>
                </div>

                <div class="field">
                    <label for="notes" class="optional">Notes</label>
                    <textarea id="notes" :maxlength="notesMaxLength" name="notes" placeholder="Notes"
                              v-model="form.notes"></textarea>
                </div>
            </fieldset>
            <fieldset>
                <legend>Styling</legend>

                <GridCustomizeTeam name="Away Team" v-model="form.awayTeam"/>
                <GridCustomizeTeam name="Home Team" v-model="form.homeTeam"/>
            </fieldset>

            <div class="buttons">
                <button type="button" class="secondary" @click.prevent="didClickCancel">Cancel</button>
                <button type="submit" name="submit">Save</button>
            </div>
        </form>
    </section>
</template>

<script>
    import GridCustomizeTeam from './GridCustomizeTeam.vue'
    import ModalController from '@/controllers/ModalController'
    import sqmgrClient from "@/models/sqmgrClient";
    import sqmgrConfig from "@/models/sqmgrConfig";

    export default {
        name: "GridCustomize",
        components: {GridCustomizeTeam},
        props: {
            token: {
                type: String,
                required: true,
            },
            pool: {
                type: Object,
                required: true,
            },
            grid: {
                type: Object,
                required: false,
            },
        },
        data() {
            return {
                ModalController,
                errors: null,
                notesMaxLength: 200,
                form: {
                    eventDate: '0000-00-00',
                    notes: '',
                    rollover: false,
                    label: '',
                    awayTeam: {
                        name: '',
                        color1: '',
                        color2: '',
                    },
                    homeTeam: {
                        name: '',
                        color1: '',
                        color2: '',
                    }
                }
            }
        },
        created() {
            sqmgrConfig()
                .then(config => this.notesMaxLength = config.notesMaxLength)
                .catch(err => ModalController.showError(err))
        },
        beforeMount() {
            if (this.grid) {
                const date = this.grid.eventDate.substr(0, 10)
                this.form.eventDate = date === '0001-01-01' ? '' : date
                this.form.notes = this.grid.settings.notes
                this.form.rollover = this.grid.rollover
                this.form.label = this.grid.label
                this.form.awayTeam.name = this.grid.awayTeamName
                this.form.awayTeam.color1 = this.grid.settings.awayTeamColor1
                this.form.awayTeam.color2 = this.grid.settings.awayTeamColor2
                this.form.homeTeam.name = this.grid.homeTeamName
                this.form.homeTeam.color1 = this.grid.settings.homeTeamColor1
                this.form.homeTeam.color2 = this.grid.settings.homeTeamColor2
            }
        },
        methods: {
            submit() {
                this.errors = null

                const id = this.grid ? this.grid.id : 0
                const data = {
                    eventDate: this.form.eventDate,
                    notes: this.form.notes,
                    label: this.form.label,
                    homeTeamName: this.form.homeTeam.name,
                    homeTeamColor1: this.form.homeTeam.color1,
                    homeTeamColor2: this.form.homeTeam.color2,
                    awayTeamName: this.form.awayTeam.name,
                    awayTeamColor1: this.form.awayTeam.color1,
                    awayTeamColor2: this.form.awayTeam.color2,
                }

                if (this.pool.gridType === 'roll100') data.rollover = this.form.rollover

                sqmgrClient.saveGrid(this.token, id, data)
                    .then(grid => {
                        this.$emit('saved', grid)
                    })
                    .catch(err => {
                        if (err.response && err.response.data && err.response.data.result) {
                            this.errors = err.response.data.result
                        }

                        ModalController.showError(err)
                    })
            },
            didClickCancel() {
                this.$emit('canceled')
                ModalController.abort()
            }
        },
    }
</script>

<style scoped lang="scss">
    section.grid-customize {
        position: relative;
        width:    70vw;
    }
</style>