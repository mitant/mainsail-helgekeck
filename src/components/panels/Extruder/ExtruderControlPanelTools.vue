<template>
    <div class="mb-3">
        <v-row v-for="(row, index) in rows" :key="'row_' + index" class="mt-0">
            <v-col>
                <v-item-group class="_btn-group py-0 px-3">
                    <extruder-control-panel-tools-item v-for="macro in row" :key="macro.name" :macro="macro" />
                </v-item-group>
            </v-col>
        </v-row>
    </div>
</template>

<script lang="ts">
import { mdiPrinter3dNozzle } from '@mdi/js'
import { Component, Mixins } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import ControlMixin from '@/components/mixins/control'

@Component({})
export default class ExtruderControlPanel extends Mixins(BaseMixin, ControlMixin) {
    mdiPrinter3dNozzle = mdiPrinter3dNozzle

    get rows() {
        const cols = 6
        let rows = []

        for (let i = 0; i < this.toolchangeMacros.length; i += cols) {
            rows.push(this.toolchangeMacros.slice(i, i + cols))
        }

        return rows
    }

    get wrappedToolchangeMacros(): any[] {
        const rows: any[] = []

        let maxCols = 6
        let rowIndex = 0
        let row: PrinterStateToolchangeMacro[] = []
        rows.push(row)
        for (const macro of this.toolchangeMacros) {
            if (rows[rowIndex].length == maxCols) {
                rowIndex = rowIndex + 1
                let row: PrinterStateToolchangeMacro[] = []
                rows.push(row)
            }
            rows[rowIndex].push(macro)
        }

        return rows
    }

    get isPrinting(): boolean {
        return ['printing'].includes(this.printer_state)
    }

    get primaryColor(): string {
        return this.$store.state.gui.uiSettings.primary
    }

    get primaryTextColor(): string {
        let splits = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(this.primaryColor)
        if (splits) {
            const r = parseInt(splits[1], 16) * 0.2126
            const g = parseInt(splits[2], 16) * 0.7152
            const b = parseInt(splits[3], 16) * 0.0722
            const perceivedLightness = (r + g + b) / 255

            return perceivedLightness > 0.7 ? '#222' : '#fff'
        }

        return '#ffffff'
    }

    get warningColor(): string {
        return this.$vuetify?.theme?.currentTheme?.warning?.toString() ?? '#ff8300'
    }

    doSend(gcode: string): void {
        this.$store.dispatch('server/addEvent', { message: gcode, type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: gcode }, { loading: gcode.toLowerCase() })
    }
}
</script>

<style scoped>
._btn-group {
    border-radius: 4px;
    display: inline-flex;
    flex-wrap: nowrap;
    max-width: 100%;
    min-width: 100%;

    .v-btn {
        border-radius: 0;
        border-color: rgba(255, 255, 255, 0.12);
        border-style: solid;
        border-width: thin;
        box-shadow: none;
        height: 28px;
        opacity: 0.8;
        min-width: auto !important;
    }

    .v-btn:first-child {
        border-top-left-radius: inherit;
        border-bottom-left-radius: inherit;
    }

    .v-btn:last-child {
        border-top-right-radius: inherit;
        border-bottom-right-radius: inherit;
    }

    .v-btn:not(:first-child) {
        border-left-width: 0;
    }
}

html.theme--light ._btn-group .v-btn {
    border-color: rgba(0, 0, 0, 0.12);
}
</style>
