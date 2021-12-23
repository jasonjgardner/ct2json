<template>
  <div class="ct">
    <div class="ct__input">
      <textarea
        v-model="input"
        rows="50"
        cols="50"
        spellcheck="false"
        @blur="parseInput"
        placeholder="Paste CheatTable XML from clipboard"
      ></textarea>
      <button class="btn" type="submit" @click.prevent="parseInput">
        Parse CheatTable XML
      </button>
    </div>
    <div class="ct__output">
      <output v-show="error" v-text="error"></output>
      <pre><code v-text="out"></code></pre>
    </div>
  </div>
</template>

<script>
import { parseStringPromise } from "xml2js";

const getLabel = (description = "") =>
  (Array.isArray(description) ? description[0] || "" : "")
    .toString()
    .trim()
    .toLowerCase()
    .replace(/^"+/, "")
    .replace(/"+$/, "");

const getEntries = (list = []) => [...(list.CheatEntries[0].CheatEntry || [])];

const getValue = ({ LastState }) => {
  if (!Array.isArray(LastState) || LastState.length < 1) {
    return null;
  }

  return +LastState[0].$.Value;
};

export default {
  name: "CheatTable",
  data: () => ({
    input: "",
    xml: "",
    out: {},
    error: null,
  }),
  methods: {
    async parseInput() {
      let cheats;

      try {
        const {
          CheatTable: { CheatEntries },
        } = await parseStringPromise(this.input);
        cheats = CheatEntries[0]?.CheatEntry;
      } catch (err) {
        this.error = err;
        console.error(`Failed parsing XML: ${err}`);
        return;
      }

      const mediaCoefficients = {};

      for (const entry of cheats) {
        const entryKey = getLabel(entry.Description);
        const medias = getEntries(entry);

        for (const media of medias) {
          const mediaKey = getLabel(media.Description);
          const channels = getEntries(media);

          const value = [0, 0, 0];

          for (const channel of channels) {
            const channelKey = getLabel(channel.Description);

            let idx = 0;

            if (channelKey.startsWith("green")) {
              idx = 1;
            }

            if (channelKey.startsWith("blue")) {
              idx = 2;
            }

            value[idx] = getValue(channel);
          }

          if (
            !Object.prototype.hasOwnProperty.call(mediaCoefficients, entryKey)
          ) {
            mediaCoefficients[entryKey] = {};
          }

          mediaCoefficients[entryKey][mediaKey] = value;
        }
      }

      this.out = JSON.stringify(mediaCoefficients, null, 2).replace(
        /"([0-9]+\.[0-9]+)"/gm,
        (m) => m.replace(/"/g, "")
      );
    },
  },
};
</script>
