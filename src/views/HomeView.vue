<template>
  <div class="home-view">
    <div class="parser-select">
      <BaseSelectVue :options="sourcesOptions" v-model="sourceId" />
    </div>
    <input
      type="text"
      class="change-key"
      placeholder="Введите ключевые слова"
      v-model="keyword"
    />
    <div v-if="loading">Loading ...</div>
    <div class="btns-wrapper">
      <button @click="createParser" class="button button__add">
        Сделать запрос
      </button>
      <button
        @click="loadInfo(this.hashParsource)"
        class="button button__refresh"
      >
        Обновить
      </button>
    </div>
    {{ updatedCardArray }}
    <div class="cards-wrapper">
      <div v-for="ad in ads" class="card__wrapper" :key="ad.id">
        <div class="card__header">
          <span
            @blur="changeCardHeader($event, ad)"
            class="card__header-title"
            contenteditable="true"
            >{{ ad.title }}</span
          >
          <img
            src="../assets/images/pen.png"
            alt="image pen"
            class="img-pen img-pen-header"
          />
        </div>
        <div class="card__content">
          <div class="card__content-text">
            <img
              src="../assets/images/pen.png"
              alt="image pen"
              class="img-pen img-pen-body"
            />
            <p
              @blur="changeCardDescription($event, ad)"
              class="card__dscr"
              contenteditable="true"
            >
              {{ ad.article }}
            </p>
          </div>
          <div class="card-btns-wrapper">
            <button
              v-if="isChanged(ad.id)"
              @click="changeCardInfo(ad.id)"
              class="button button__card-send"
            >
              Сохранить изменения
            </button>
            <button @click="deleteCard(ad.id)" class="button button__card">
              Удалить
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";
import BaseSelectVue from "@/components/BaseSelect.vue";

export default {
  name: "HomeView",
  components: { BaseSelectVue },
  data() {
    return {
      keyword: "",
      updateAvailable: false,
      loading: false,
      sourceId: null,
      hashParsource: "",
      sourcesOptions: [
        { title: "выбрать всё", id: 0, code: "all" },
        { title: "freelance", id: 1, code: "freelancehabr" },
        { title: "kwork", id: 2, code: "kworkservice" },
      ],
      ads: [],
      parserName: "",
      parserResponse: {},
      cardsId: [],
      updatedCardArray: [],
    };
  },
  computed: {
    currentTypeName() {
      let result = "all";
      if (this.sourceId && this.sourceId !== 0) {
        let currentType = this.sourcesOptions.find(
          (item) => item.id === this.sourceId
        );
        result = currentType.code + "_result";
      }
      return result;
    },
  },

  methods: {
    isChanged(id) {
      return !!this.updatedCardArray.find((i) => i.id === id);
    },
    generateHashParsource() {
      return Math.random() + "fff";
    },
    changeCardHeader(e, item) {
      let updatedItem = { ...item, title: e.target.innerHTML };
      this.updatedCardArray.push(updatedItem);
    },
    changeCardDescription(e, item) {
      let updatedItem = { ...item, description: e.target.innerHTML };
      this.updatedCardArray.push(updatedItem);
    },
    async changeCardInfo(id) {
      const endpoint = this.defineSource(id);
      let item = this.updatedCardArray.find((i) => i.id === id);
      try {
        await axios.put(
          `https://parser.api-compas-goo.ru/api/${endpoint}/${id}/`,
          {
            ...item,
          },
          {
            headers: {
              Authorization:
                "Api-Key VSfHLDHK.MM3MeLZNV9HMDEY5s1zz4Yl79lbRwoux",
            },
          }
        );
      } catch (err) {
        console.err(err);
        throw err;
      }
    },
    async createParser() {
      this.hashParsource = this.generateHashParsource();
      this.loading = true;
      let sourcesIds = [];
      if (this.sourceId !== 0) {
        sourcesIds = [this.sourceId];
      } else {
        let sourcesOptionsIds = this.sourcesOptions.map((i) => i.id);
        sourcesIds = sourcesOptionsIds.slice(1);
      }
      try {
        const response = await axios.post(
          "https://parser.api-compas-goo.ru/parser-config/",
          {
            sources: sourcesIds,
            id_user: 0,
            id_parsource: 0,
            keywords: this.keyword,
            hash_parsource: this.hashParsource,
          },
          {
            headers: {
              Authorization:
                "Api-Key VSfHLDHK.MM3MeLZNV9HMDEY5s1zz4Yl79lbRwoux",
            },
          }
        );
        if (response.status === 201) {
          this.updateAvailable = true;
          this.loading = false;
        }
      } catch (err) {
        this.loading = false;
        this.updateAvailable = false;
        console.err(err);
        throw err;
      }
    },
    async loadInfo(hash) {
      try {
        const response = await axios.get(
          "https://parser.api-compas-goo.ru/parser-config/",
          {
            params: {
              hash_parsource: hash,
            },
            headers: {
              Authorization:
                "Api-Key VSfHLDHK.MM3MeLZNV9HMDEY5s1zz4Yl79lbRwoux",
            },
          }
        );
        if (response.status === 200) {
          this.parserResponse = response.data[0];
          if (this.currentTypeName === "all") {
            this.ads = response.data[0].kwork_result.concat(
              response.data[0].freelancehabr_result
            );
          } else {
            this.ads = response.data[0][this.currentTypeName];
          }
        }
      } catch (err) {
        console.err(err);
        throw err;
      }
    },
    defineSource(id) {
      let keys = [
        { source: "freelancehabr_result", endpoint: "freelancehabr" },
        { source: "kwork_result", endpoint: "kworkservice" },
      ];
      let result = "";
      keys.forEach((el) => {
        let item = this.parserResponse[el.source].find((i) => i.id === id);
        if (item) {
          result = el.endpoint;
        }
      });
      return result;
    },
    async deleteCard(id) {
      this.loading = true;
      const endpoint = this.defineSource(id);
      try {
        const response = await axios.delete(
          `https://parser.api-compas-goo.ru/api/${endpoint}/${id}`,
          {
            headers: {
              Authorization:
                "Api-Key VSfHLDHK.MM3MeLZNV9HMDEY5s1zz4Yl79lbRwoux",
            },
          }
        );
        if (response.status === 204) {
          this.loading = false;
          await this.loadInfo(this.hashParsource);
        }
      } catch (err) {
        this.loading = false;
        console.err(err);
        throw err;
      }
    },
  },
};
</script>
<style scoped lang="scss">
.home-view {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.card__wrapper {
  width: 270px;
  max-height: 270px;
  overflow-y: auto;
  border: 1px solid #61affe;
  background: rgba(97, 175, 254, 0.1);
  border-radius: 4px;
  box-shadow: 0 0 3px rgb(0 0 0 / 19%);
  padding: auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.card__header {
  text-align: center;
  padding-top: 15px;
  padding-bottom: 15px;
  font-weight: bold;
}
.card__content-text {
  text-align: left;
  padding-left: 15px;
}
.card__dscr {
  max-width: 230px;
}
.btns-wrapper {
  margin: 15px auto;
}
.parser-select {
  width: 300px;
}

.change-key {
  width: 270px;
  height: 38px;
  border: 1px solid #61affe;
  background: rgba(97, 175, 254, 0.1);
  border-radius: 4px;
  box-shadow: 0 0 3px rgb(0 0 0 / 19%);

  &::-webkit-input-placeholder {
    /* WebKit browsers */
    padding-left: 12px;
  }
  &:-moz-placeholder {
    /* Mozilla Firefox 4 to 18 */
    padding-left: 12px;
  }
  &::-moz-placeholder {
    /* Mozilla Firefox 19+ */
    padding-left: 12px;
  }
  &:-ms-input-placeholder {
    /* Internet Explorer 10+ */
    padding-left: 12px;
  }
}

.cards-wrapper {
  margin-top: 25px;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: 1fr;
  gap: 25px;
  font-size: 12px;
}
.card-btns-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.button {
  width: 130px;
  height: 38px;
  font-size: 14px;
  border: 1px solid #61affe;
  background: rgba(97, 175, 254, 0.1);
  border-radius: 4px;
  box-shadow: 0 0 3px rgb(0 0 0 / 19%);
  font-weight: bold;

  &__add {
    margin-right: 10px;
    border-color: #50e3c2;
    background: rgba(80, 227, 194, 0.1);
  }

  &__refresh {
    border-color: #fca130;
    background: rgba(252, 161, 48, 0.1);
  }
  &__card {
    text-align: center;
    margin-top: 15px;
    margin-bottom: 15px;
  }
  &__card-send {
    font-size: 12px;
  }
}

.card__header {
  display: flex;
  flex-direction: row;
  align-items: center;
  // justify-content: center;
}

.card__header-title {
  white-space: normal;
  overflow: hidden;
  max-width: 236px;
}
.img-pen {
  height: 10px;
  width: 10px;
  margin-left: 10px;
  display: none;
}
.card__header:hover .img-pen-header {
  display: inline;
}
.card__content-text:hover .img-pen-body {
  display: inline;
}
</style>
