<template>
    <v-container class="py-0">
        <v-item-group class="_btn-group py-0">
            <v-btn
                :class="idexMode != 'copy' && idexMode != 'mirror' ? 'primary--text' : {}"
                :disabled="isPrinting || !homedAxes.includes('xyz')"
                :style="{
                    'background-color': idexMode != 'copy' && idexMode != 'mirror' ? 'var(--color-primary)' : '',
                    color: idexMode != 'copy' && idexMode != 'mirror' ? 'primary' : '',
                }"
                dense
                class="_btn-qs flex-grow-1 px-1"
                @click="doSend('IDEX_SINGLE')">
                SINGLE MODE
            </v-btn>
            <v-btn
                :class="idexMode == 'copy' ? 'primary--text' : {}"
                :disabled="isPrinting || !homedAxes.includes('xyz')"
                :style="{
                    'background-color': idexMode == 'copy' ? 'var(--color-primary)' : '',
                    color: idexMode == 'copy' ? 'primary' : '',
                }"
                dense
                class="_btn-qs flex-grow-1 px-1"
                @click="doSend('IDEX_COPY')">
                {{idexMode}}
            </v-btn>
            <v-btn
                :class="idexMode == 'mirror' ? 'primary--text' : {}"
                :disabled="isPrinting || !homedAxes.includes('xyz')"
                :style="{
                    'background-color': idexMode == 'mirror' ? 'var(--color-primary)' : '',
                    color: idexMode == 'mirror' ? 'primary' : '',
                }"
                dense
                class="_btn-qs flex-grow-1 px-1"
                @click="doSend('IDEX_MIRROR')">
                MIRROR MODE
            </v-btn>
        </v-item-group>
    </v-container>
</template>

<script lang="ts">
import { Component, Mixins, Watch } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import ControlMixin from '@/components/mixins/control'

@Component
export default class IdexControl extends Mixins(BaseMixin, ControlMixin) {

    get isPrinting() {
        return ['printing'].includes(this.printer_state)
    }

    get homedAxes(): string {
        return this.$store.state.printer?.toolhead?.homed_axes ?? ''
    }

    get isIDEX(): boolean {
        try {
            const dualCarriage = this.$store.state.printer.configfile?.settings?.dual_carriage
            if (dualCarriage) return true
            return false
        } catch {
            return false
        }
    }

    get idexMode(): string {
        try {
            const dualCarriage = this.$store.state.printer.configfile?.settings?.dual_carriage
            return dualCarriage?.carriage_1?.toString().toLowerCase()
        } catch {
            return 'error'
        }
    }

    doSend(gcode: string): void {
        this.$store.dispatch('server/addEvent', { message: gcode, type: 'command' })
        this.$socket.emit('printer.gcode.script', { script: gcode }, { loading: gcode.toLowerCase() })
    }
}
</script>

<style lang="scss" scoped>
.v-btn-toggle {
    width: 100%;
}

._btn-group {
    border-radius: 4px;
    display: inline-flex;
    flex-wrap: nowrap;
    max-width: 100%;
    min-width: 100%;
    width: 100%;

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

._btn-qs {
    font-size: 0.8rem !important;
    font-weight: 400;
    max-height: 28px;
}
</style>