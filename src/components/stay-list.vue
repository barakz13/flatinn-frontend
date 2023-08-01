<template>
  <h4 v-if="stays.length" class="stay-list-title">
    {{
      $route.query.address
        ? `${stays.length} stays in ${$route.query.address}`
        : 'Explore the world!'
    }}
  </h4>
  <h4 v-else>No match found</h4>

  <!-- <h4 v-if="$route.query.address" class="stay-list-title">
    {{ stays.length }} stays in
    {{ stays[0].address.city }} 
  </h4>
  <h4 v-else-if="!stays.length">Explore the world!</h4>
  <h4 v-else>No match found</h4> -->
  <br />
  <!-- Buttons for additional filtters -->
  <!-- <expolore-btns class="explore-btns" /> -->
  <section class="explore-btns">
    <button
      @click.stop.prevent="togglePrice"
      class="explore-btn"
      :class="{ clicked: isPriceActive }"
    >
      Price
    </button>
    <button
      @click.stop.prevent="toggleType"
      class="explore-btn"
      :class="{ clicked: isTypeActive }"
    >
      Type of place
    </button>
    <button
      @click.stop.prevent="toggleWifi"
      class="explore-btn"
      :class="{ clicked: isWifiActive }"
    >
      Wifi
    </button>
    <button
      @click.stop.prevent="toggleTv"
      class="explore-btn"
      :class="{ clicked: isTvActive }"
    >
      TV
    </button>
    <button
      @click.stop.prevent="toggleKitchen"
      class="explore-btn"
      :class="{ clicked: isKitchenActive }"
    >
      Kitchen
    </button>
    <button
      @click.stop.prevent="toggleAc"
      class="explore-btn"
      :class="{ clicked: isAcActive }"
    >
      AC
    </button>
    <button
      @click.stop.prevent="toggleSmoking"
      class="explore-btn"
      :class="{ clicked: isSmokingAllowedActive }"
    >
      Smoking allowed
    </button>
    <button
      @click.stop.prevent="togglePets"
      class="explore-btn"
      :class="{ clicked: isPetsAllowedActive }"
    >
      Pets allowed
    </button>

    <div v-if="isPriceActive" class="price-modal">
      <div class="range-input-container">
        <label class="range-input-label">
          Min:
          <input
            class="range-input input-min"
            type="number"
            placeholder="minimum price"
            v-model="priceRange.min"
          />
        </label>
        <label class="range-input-label">
          Max:
          <input
            class="range-input input-max"
            type="number"
            placeholder="maximum price"
            v-model="priceRange.max"
          />
        </label>
      </div>
      <div class="action-buttons">
        <button class="clear-btn" @click.stop.prevent="clearPrice">
          Clear
        </button>
        <button class="save-btn" @click.stop.prevent="setPrice">Save</button>
      </div>
    </div>
    <div v-if="isTypeActive" class="type-room-modal">
      <div class="room-options">
        <label @click="toggleEntire" class="room-options-label">
          <div class="room-options-input-div">
            <span class="room-options-main-span">
              <span class="room-options-input-span span-off">
                <span
                  v-if="isEntireChecked"
                  class="room-options-input-span span-on"
                >
                  <img
                    src="../assets/checkmark-filter.svg"
                    alt="room-options-checkmark"
                    class="room-options-checkmark"
                  />
                </span>
              </span>
            </span>
          </div>
          <div class="room-options-txt">
            <h5 class="room-options-title">Entire Home/Apt</h5>
            <h5 class="room-options-subtitle">A room to yourself</h5>
          </div>
        </label>
        <label @click="togglePrivate" class="room-options-label">
          <div class="room-options-input-div">
            <span class="room-options-main-span">
              <span class="room-options-input-span span-off">
                <span
                  v-if="isPrivateChecked"
                  class="room-options-input-span span-on"
                >
                  <img
                    src="../assets/checkmark-filter.svg"
                    alt="room-options-checkmark"
                    class="room-options-checkmark"
                  />
                </span>
              </span>
            </span>
          </div>
          <div class="room-options-txt">
            <h5 class="room-options-title">Private Room</h5>
            <h5 class="room-options-subtitle">
              Your own room in a home or a hotel, plus some shared common spaces
            </h5>
          </div>
        </label>
      </div>
      <div class="action-buttons">
        <button class="clear-btn" @click.stop.prevent="clearRoomType">
          Clear
        </button>
        <button class="save-btn" @click.stop.prevent="setRoom">Save</button>
      </div>
    </div>
  </section>
  <div v-if="stays.length === 0">
    <div class="full-body">
      <div class="load-img-con">
        <img src="../assets/img/loading.gif" alt="Loading" />
      </div>
    </div>
  </div>
  <ul v-else class="stay-list">
    <stay-preview v-for="stay in stays" :stay="stay" :key="stay._id" />
  </ul>
</template>

<script>
import stayPreview from './stay-preview.vue';
import expoloreBtns from './explore-btns.vue';

export default {
  props: {
    stays: {
      type: Array,
      required: true,
    },
  },
  emits: ['btnsFilter'],
  data() {
    return {
      priceRange: { min: 0, max: 100000 },
      isPrivateChecked: false,
      isEntireChecked: false,
      isPriceActive: false,
      isTypeActive: false,
      // Amenities
      isWifiActive: false,
      isTvActive: false,
      isKitchenActive: false,
      isAcActive: false,
      isPetsAllowedActive: false,
      isSmokingAllowedActive: false,
      exploreBtnsFilter: {
        amenities: [],
        roomType: [],
        priceRange: {
          min: -Infinity,
          max: Infinity,
        },
      },
    };
  },
  created() {
    // console.log(this.$route.query);
    console.log('LIST CREATED!!!===================================');
    this.getPriceRange();
  },
  components: {
    stayPreview,
    expoloreBtns,
  },
  methods: {
    toggleEntire() {
      this.isEntireChecked = !this.isEntireChecked;
      console.log('this.isEntireChecked', this.isEntireChecked);
      if (this.isEntireChecked)
        this.exploreBtnsFilter.roomType.push('Entire home/apt');
      else {
        const idx = this.exploreBtnsFilter.roomType.findIndex(
          (roomType) => roomType === 'Entire home/apt'
        );
        this.exploreBtnsFilter.amenities.splice(idx, 1);
      }
    },
    togglePrivate() {
      this.isPrivateChecked = !this.isPrivateChecked;
      console.log('this.isPrivateChecked', this.isPrivateChecked);
      if (this.isPrivateChecked)
        this.exploreBtnsFilter.roomType.push('private room');
      else {
        const idx = this.exploreBtnsFilter.roomType.findIndex(
          (roomType) => roomType === 'Private room'
        );
        this.exploreBtnsFilter.amenities.splice(idx, 1);
      }
    },
    togglePrice() {
      this.isPriceActive = !this.isPriceActive;
      this.isTypeActive = false;
    },
    toggleType() {
      this.isTypeActive = !this.isTypeActive;
      this.isPriceActive = false;
    },

    toggleWifi() {
      this.isWifiActive = !this.isWifiActive;
      this.setAmenities('Wifi', this.isWifiActive);
    },
    toggleTv() {
      this.isTvActive = !this.isTvActive;
      this.setAmenities('TV', this.isTvActive);
    },
    toggleKitchen() {
      this.isKitchenActive = !this.isKitchenActive;
      this.setAmenities('Kitchen', this.isKitchenActive);
    },
    toggleAc() {
      this.isAcActive = !this.isAcActive;
      this.setAmenities('Air conditioning', this.isAcActive);
    },
    toggleSmoking() {
      this.isSmokingAllowedActive = !this.isSmokingAllowedActive;
      this.setAmenities('Smoking allowed', this.isSmokingAllowedActive);
    },
    togglePets() {
      this.isPetsAllowedActive = !this.isPetsAllowedActive;
      this.setAmenities('Pets allowed', this.isPetsAllowedActive);
    },

    setAmenities(name, btnStatus) {
      if (btnStatus) {
        if (this.exploreBtnsFilter.amenities.includes(name)) return;
        this.exploreBtnsFilter.amenities.push(name);
      } else {
        const idx = this.exploreBtnsFilter.amenities.findIndex(
          (amenity) => amenity === name
        );
        this.exploreBtnsFilter.amenities.splice(idx, 1);
      }
      this.$emit('btnsFilter', this.exploreBtnsFilter);
    },

    getPriceRange() {
      let allStays = this.$store.getters.getStaysAll;
      const prices = allStays.map((stay) => stay.price);
      this.priceRange.min = Math.min(...prices);
      console.log(' priceRange.min', this.priceRange.min);
      this.priceRange.max = Math.max(...prices);
      console.log(' priceRange.max', this.priceRange.max);
    },
    setPrice() {
      this.exploreBtnsFilter.priceRange.max = this.priceRange.max;
      this.exploreBtnsFilter.priceRange.min = this.priceRange.min;
      this.$emit('btnsFilter', this.exploreBtnsFilter);
      this.isPriceActive = false;
    },
    clearPrice() {
      this.exploreBtnsFilter.priceRange.min = -Infinity;
      this.exploreBtnsFilter.priceRange.max = Infinity;
      // this.$emit('btnsFilter', this.exploreBtnsFilter);
      this.getPriceRange();
    },

    setRoom() {
      this.$emit('btnsFilter', this.exploreBtnsFilter);
      this.isTypeActive = false;
    },
    // Like airBnb
    clearRoomType() {
      this.exploreBtnsFilter.roomType = [];
      this.isPrivateChecked = false;
      this.isEntireChecked = false;
      // this.$emit('btnsFilter', this.exploreBtnsFilter);
    },
  },
  watch: {
    // whenever question changes, this function will run
    stays: function (oldProp, newProp) {
      if (
        this.exploreBtnsFilter.priceRange.max === Infinity &&
        this.exploreBtnsFilter.priceRange.min === -Infinity
      ) {
        console.log('update get price range.........');
        this.getPriceRange();
      }
    },
    // isPriceActive() {
    //   this.arrowSign();
    // },
    // isTypeActive() {
    //   this.arrowSign();
    // },
  },
  computed: {
    //   arrowSign() {
    //     if (this.isPriceActive || this.isTypeActive) {
    //       return hi;
    //     } else !this.isPriceActive || !this.isTypeActive;
    //   },
  },
};
</script>
<style scoped>
.full-body {
  height: calc(100vh - 150px);
  width: 100%;
}
.load-img-con {
  /* margin: 0 auto; */
  margin: auto;
  margin-top: 100px;
  /* background-color: yellow; */
  width: fit-content;
  /* top: 35%; */
  /* position: absolute; */
}
</style>
