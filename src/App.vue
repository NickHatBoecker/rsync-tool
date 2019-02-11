<template>
    <v-app>
        <v-container>
            <v-form v-model="formIsValid">
                <v-container>
                    <v-layout>
                        <v-flex md4>
                            <v-select
                                v-model="method"
                                :items="methods"
                                @input="generateCommand()"
                            ></v-select>
                        </v-flex>

                        <v-flex md4>
                            <v-text-field
                                v-model="source"
                                label="Source"
                                required
                                @input="generateCommand()"
                            ></v-text-field>
                        </v-flex>

                        <v-flex md1 class="text-md-center" style="margin-top: 30px;">
                            to
                        </v-flex>

                        <v-flex xs12 md4>
                            <v-text-field
                                    v-model="target"
                                    label="Target"
                                    required
                                    @input="generateCommand()"
                            ></v-text-field>
                        </v-flex>
                    </v-layout>

                    <v-layout row wrap>
                        <v-flex xs4>
                            <v-checkbox v-model="syncSourceDirectoryContentOnly" label="Sync source directory content only" @change="generateCommand()"></v-checkbox>
                        </v-flex>
                        <v-flex xs4>
                            <v-checkbox v-model="useUpdate" label="Skip files that are newer on the receiver (-u)" @change="generateCommand()"></v-checkbox>
                        </v-flex>
                    </v-layout>
                    <v-layout row wrap>
                        <v-flex xs4>
                            <v-checkbox v-model="useArchive" label="Archive-mode (-a)" @change="generateCommand()"></v-checkbox>
                        </v-flex>
                        <v-flex xs4>
                            <v-checkbox v-model="useVerbose" label="Verbose (-v)" @change="generateCommand()"></v-checkbox>
                        </v-flex>
                        <v-flex xs4>
                            <v-checkbox v-model="compressFiles" label="Compress file data during the transfer (-z)" @change="generateCommand()"></v-checkbox>
                        </v-flex>
                    </v-layout>
                    <v-layout>
                        <v-flex xs12>
                            <v-checkbox
                                v-model="useDryRun"
                                :label="`Dry run (nothing will be ${method == 'copy' ? 'copied' : 'moved'}, --dry-run)`"
                                @change="generateCommand()"
                            ></v-checkbox>
                        </v-flex>
                    </v-layout>

                    <v-textarea
                        box
                        v-model="generatedCommand"
                        label="Generated Code"
                        read-only
                    ></v-textarea>
                </v-container>
            </v-form>
        </v-container>
    </v-app>
</template>

<script>
export default {
    name: 'App',
    data: function () {
        return {
            methods: [
                { text:'Copy', value: 'copy' },
                { text:'Move', value: 'move' },
            ],

            method: 'copy',
            source: '',
            target: '',
            generatedCommand: '',

            syncSourceDirectoryContentOnly: false,
            useDryRun: false,
            useUpdate: false,
            useArchive: true,
            useVerbose: true,
            compressFiles: true,

            formIsValid: false,
        }
    },

    methods: {
        generateCommand () {
            if (this.source === '' || this.target === '') {
                return
            }

            let source = this.source.replace(/\/$/, "")
            if (this.syncSourceDirectoryContentOnly) {
                source = source+'/'
            }

            let command = `rsync ${source} ${this.target} -${this.getOptions()}`

            if (this.method === 'move') {
                command += ' --remove-source-files'
            }
            if (this.useDryRun) {
                command += ' --dry-run'
            }

            this.generatedCommand = command
        },

        /**
         * Options from https://www.computerhope.com/unix/rsync.htm
         *
         * @returns {string}
         */
        getOptions () {
            let options = ''

            if (this.useArchive) {
                options += 'a'
            }
            if (this.useVerbose) {
                options += 'v'
            }            if (this.compressFiles) {
                options += 'z'
            }
            if (this.useUpdate) {
                options += 'u'
            }

            return options
        },
    },
}
</script>
