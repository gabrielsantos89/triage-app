<template>
  <div class="simple-keyboard"></div>
</template>

<script>
import Keyboard from 'simple-keyboard';

export default {
  name: 'SimpleKeyboard',
  props: {
    value: {
      type: String
    },
    layout: {
      type: Object
    },
    layoutName: {
      type: String,
      default: 'default'
    },
    format: {
      type: String,
      default: 'text'
    }
  },
  data: () => ({
    keyboard: null
  }),
  mounted() {
    this.keyboard = new Keyboard('.simple-keyboard', {
      layout: this.layout,
      input: this.value,
      onChange: this.onChange,
      onKeyPress: this.onKeyPress,
      layoutName: this.layoutName,
      display: {
      '{enter}': 'Enter',
      '{shift}': 'Shift',
      '{backspace}': '⌫',
      '{space}': 'Espaço'
    }
    });
  },
  methods: {
    onChange(input) {
      let formattedInput = input;
      if (this.format === 'cpf') {
        formattedInput = this.formatCPF(input);
        if (this.keyboard.getInput() !== formattedInput) {
          this.keyboard.setInput(formattedInput);
        }
      }
      this.$emit('input', input); 
    },
    onKeyPress(button) {
      this.$emit('onKeyPress', button);
      if (button === '{enter}') {
      this.$emit('onEnter');
      }   
      if (button === '{shift}' || button === '{lock}') {
        this.handleShift();
      }
    },
    handleShift() {
      let currentLayout = this.keyboard.options.layoutName;
      let shiftToggle = currentLayout === 'default' ? 'shift' : 'default';
      
      this.keyboard.setOptions({
        layoutName: shiftToggle
      });
    },
    formatCPF (value) {
      if (!value) return ""
      // 1. Remove tudo o que não for dígito
      let v = value.replace(/[^\d]/g, '')
      // 2. Limita aos 11 dígitos
      v = v.substring(0, 11)

      // 3. Aplica a máscara sequencialmente (esta lógica é mais segura)
      // (123)4... -> 123.4...
      v = v.replace(/(\d{3})(\d)/, '$1.$2')
      // (123.456)7... -> 123.456.7...
      v = v.replace(/(\d{3})\.(\d{3})(\d)/, '$1.$2.$3')
      // (123.456.789)1... -> 123.456.789-1...
      v = v.replace(/(\d{3})\.(\d{3})\.(\d{3})(\d)/, '$1.$2.$3-$4')

      return v
    }
  },
  watch: {
    value(newValue) {
      if (this.keyboard && newValue !== this.keyboard.getInput()) {
        this.keyboard.setInput(newValue);
      }
    },
    layout(newLayout) {
      if (this.keyboard) {
        this.keyboard.setOptions({
          layout: newLayout
        });
        this.keyboard.setInput(this.value || '');
      }
    },
    layoutName(newLayoutName) {
      if (this.keyboard && this.keyboard.options.layoutName !== newLayoutName) {
        this.keyboard.setOptions({
          layoutName: newLayoutName
        });
      }
    }
  },

  beforeDestroy() {
    if (this.keyboard) {
      this.keyboard.destroy();
      }
    }
}
</script>

<style lang="css">
/* Importa o CSS global do teclado.
  Usamos 'lang="css"' aqui para evitar problemas com o SASS
  ao importar um ficheiro .css.
*/
@import '~simple-keyboard/build/css/index.css';

/* Ajustes de tamanho para o totem */
.simple-keyboard {
  width: 100%;
  max-width: 1400px;
  margin: 0 auto;
  font-size: 2.2rem;
}
</style>