<template>
  <div class="save-input">
    <button @click="openFileBrowser">Browse the file...</button>
    <input type="file" ref="ipFile" @change="readFile" />
    <input
      type="text"
      v-model="fileData"
      placeholder="...or paste your save game here"
      aria-label="Text Input"
    />
  </div>
</template>

<script>
import { inflate, inflateRaw } from 'pako/lib/inflate';

const ANTI_CHEAT_CODE = 'Fe12NAfA3R6z4k0z';
const ZLIB_HEADER = '7a990d405d2c6fb93aa8fbb0ec1a3b23';
const DEFLATE_HEADER = '7e8bb5a89f2842ac4af01b3b7e228592';

export default {
  name: 'SaveInput',
  data: () => ({
    fileData: '',
    decodedData: '',
  }),
  watch: {
    fileData: function(newValue, oldValue) {
      if (newValue !== oldValue) {
        try {
          this.decodedData =
            newValue.substring(0, ZLIB_HEADER.length) === ZLIB_HEADER
              ? this.decodeZLib(newValue)
              : newValue.substring(0, DEFLATE_HEADER.length) === DEFLATE_HEADER
              ? this.decodeDeflate(newValue)
              : this.decodeBase64(newValue);
        } catch (e) {
          this.decodedData = '';
          // console.log('Save file corrupted and unable to read!');
          console.log(e);
        }
      }
    },
    decodedData: function(newValue) {
      if (newValue !== '') {
        this.$store.dispatch('setRawData', newValue);
      }
    },
  },
  methods: {
    openFileBrowser() {
      this.$refs.ipFile.click();
    },
    decodeZLib(encoded) {
      let output = atob(encoded.substring(ZLIB_HEADER.length));
      output = JSON.parse(inflate(output, { to: 'string' }));
      return output;
    },
    decodeDeflate(encoded) {
      let output = atob(encoded.substring(DEFLATE_HEADER.length));
      output = JSON.parse(inflateRaw(output, { to: 'string' }));
      return output;
    },
    decodeBase64(encoded) {
      let output = '';
      const encodedHasAntiCheat = encoded.indexOf(ANTI_CHEAT_CODE);
      if (encodedHasAntiCheat) {
        output = encoded.substr(encodedHasAntiCheat);
        output = [...output].filter((_, i) => i % 2 === 1).join('');
        output = JSON.parse(atob(output));
      } else {
        output = JSON.parse(encoded);
      }
      return output;
    },
    readFile(e) {
      const _fileReader = new FileReader();
      _fileReader.onload = (e) => {
        this.fileData = e.target.result;
      };
      _fileReader.readAsText(e.target.files[0]);
    },
  },
};
</script>

<style scoped lang="scss">
.save-input {
  display: flex;
  align-items: center;
  justify-content: center;

  > button {
    flex: 0 0 250px;
    margin: 10px;
    border: 0;
    border-radius: 4px;
    padding: 20px 0;
    background-color: var(--btn-bg-color);
    font-size: 16px;
    color: var(--color);

    &:hover {
      background-color: var(--btn-bg-color-hover);
    }
  }

  > input {
    &[type='file'] {
      display: none;
    }

    &[type='text'] {
      flex: 0 0 250px;
      display: block;
      margin: 10px;
      border: 1px solid var(--color);
      border-radius: 4px;
      padding: 19px 10px;
      background-color: var(--bg-color);
      font-size: 16px;
      color: var(--color);
    }
  }
}

@media screen and (max-width: 525px) {
  .save-input {
    display: block;
    padding: 10px;

    > button {
      width: 100%;
      margin: 0;
    }

    > input {
      &[type='text'] {
        width: 100%;
        margin: 10px 0 0;
      }
    }
  }
}
</style>
