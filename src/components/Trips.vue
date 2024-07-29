<template>
  <section>
    <header class="bg-gray-800 text-white py-4">
      <div class="container mx-auto px-4 text-center">
        <h1 class="text-3xl font-semibold">Select your bus type</h1>
      </div>
    </header>
    <div class="container mx-auto px-4 py-8">
      <div v-if="filteredTrips.length" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
        <div v-for="trip in filteredTrips" :key="trip.trip_id" class="bg-white rounded-lg shadow-md">
          <div class="p-4">
            <img :src="getVehicleImage(trip.vehicleType)" alt="Vehicle" class="w-full h-auto mb-4" />
          </div>
          <div class="px-4 pb-4">
            <h2 class="text-lg font-semibold">{{ trip.destination }}</h2>
            <p class="text-sm text-gray-500 mb-2">Departure: {{ trip.departure }}</p>
            <p class="text-sm text-gray-500 mb-2">Terminal: {{ trip.terminal }}</p>
            <p class="text-sm text-gray-500 mb-2">Date: {{ trip.date }}</p>
            <p class="text-sm text-gray-500 mb-2">Seats Left: {{ trip.seatsLeft }} / {{ trip.numberOfSeats }}</p>
            <p class="text-sm text-gray-500 mb-2">Vehicle Type: {{ trip.vehicleType }}</p>
            <p class="text-sm text-gray-500 mb-2">Price: ${{ trip.price }}</p>
            <div class="flex items-center mb-2">
              <input type="checkbox" v-model="trip.roundTrip" @change="toggleReturnDate(trip)" class="form-checkbox h-5 w-5 text-indigo-600" />
              <label for="roundTrip" class="ml-2 text-sm text-gray-500">To and Fro</label>
            </div>
            <input type="date" v-if="trip.roundTrip && trip.showReturnDate" v-model="trip.returnDate" @input="formatReturnDate(trip)" class="border border-gray-300 rounded-md px-3 py-2 w-full" />
          </div>
          <div class="px-4 pb-4">
            <button v-if="trip.seatsLeft > 0" class="font-bold rounded-lg text-base w-full bg-[#374151] text-white py-2" @click="bookTrip(trip)">Book Trip</button>
            <div v-else class="text-red-500 text-center">No seats available for this trip</div>
          </div>
        </div>
      </div>
      <div v-else class="text-center text-gray-500">No trips available</div>
    </div>
  </section>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      trips: [],
      user: {
        user_id: null,
        email: '',
        token: ''
      }
    };
  },
  computed: {
    filteredTrips() {
      const { destination, departure } = this.$route.query;
      return this.trips.filter(trip => {
        return (!destination || trip.destination.toLowerCase() === destination.toLowerCase()) &&
               (!departure || trip.departure.toLowerCase() === departure.toLowerCase());
      });
    }
  },
  methods: {
    getVehicleImage(vehicleType) {
      const vehicleImages = {
        Bus: 'https://st2.depositphotos.com/1358992/43886/i/450/depositphotos_438868626-stock-photo-white-minibus-isolated-white-background.jpg',
        'Luxurious Bus': 'https://st2.depositphotos.com/1358992/43886/i/450/depositphotos_438868626-stock-photo-white-minibus-isolated-white-background.jpg',
        Travel: '/images/travel.jpg'
      };
      return vehicleImages[vehicleType] || '/images/default.jpg';
    },
    toggleReturnDate(trip) {
      console.log('Before toggle:', trip.price, trip.roundTrip);
      if (trip.roundTrip) {
        trip.showReturnDate = true;
        trip.price = trip.price * 2;
      } else {
        trip.showReturnDate = false;
        trip.price = trip.price / 2;
        trip.returnDate = null;
      }
      console.log('After toggle:', trip.price);
    },
    formatReturnDate(trip) {
      if (trip.returnDate) {
        const date = new Date(trip.returnDate);
        trip.returnDate = date.toISOString().split('T')[0];
      }
    },
async bookTrip(trip) {
  try {
    const user_id = localStorage.getItem('user_id');
    if (trip.seatsLeft <= 0) {
      console.error('No seats available for this trip');
      return;
    }
    
    // Update price for round trip
    if (trip.roundTrip) {
      trip.price *= 2; // Double the price for round trip
    } else {
      trip.price /= 2; // Halve the price for one-way trip
    }

    // Save details to local storage
    localStorage.setItem('price', trip.price); // Save the doubled/adjusted price
    localStorage.setItem('trip_id', trip.trip_id);
    localStorage.setItem('user_id', user_id);
    localStorage.setItem('isRoundTrip', trip.roundTrip);
    if (trip.roundTrip) {
      localStorage.setItem('returnTrip', JSON.stringify({
        destination: trip.destination,
        departure: trip.departure,
        time: trip.time,
        price: trip.price / 2, // Store only the price for one way
        returnDate: trip.returnDate
      }));
    }

    const response = await axios.post('https://busbooking-eyow.onrender.com/api/v1/booking/book-trip', {
      trip_id: trip.trip_id,
      user_id: user_id,
      isRoundTrip: trip.roundTrip,
      returnTrip: trip.roundTrip ? {
        destination: trip.destination,
        departure: trip.departure,
        time: trip.time,
        price: trip.price / 2, // Ensure price for return trip is correctly set
        returnDate: trip.returnDate
      } : null
    });

    if (response.data.success) {
      const { authorization_url, reference } = response.data.data.payment;
      window.location.href = authorization_url;

      localStorage.setItem('paymentReference', reference);
    } else {
      console.error('Failed to initiate booking');
    }
  } catch (error) {
    console.error('Error booking trip:', error);
  }
}

,
    async fetchTrips() {
      try {
        const response = await axios.get('https://busbooking-eyow.onrender.com/api/v1/booking/alltrip');
        if (response.data.success) {
          this.trips = response.data.data;
        } else {
          console.error('Failed to fetch trips');
        }
      } catch (error) {
        console.error('Error fetching trips:', error);
      }
    }
  },
  mounted() {
    this.user.user_id = localStorage.getItem('user_id');
    this.fetchTrips();
    const urlParams = new URLSearchParams(window.location.search);
    const reference = urlParams.get('reference');
    if (reference) {
      this.verifyBooking();
    }
  }
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Roboto+Slab:wght@100..900&display=swap");

p,
h1,
h2,
h3,
h4,
h5,
h6 {
  font-family: "Roboto Slab", sans-serif;
}
</style>




