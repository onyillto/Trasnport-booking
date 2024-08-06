<template>
  <section>
    <header class="bg-gray-800 text-white py-4">
      <div class="container mx-auto px-4 text-center">
        <h1 class="text-3xl font-semibold">Select your bus type</h1>
      </div>
    </header>
    <div class="container mx-auto px-4 py-8">
      <!-- Notification Message -->
      <div v-show="notification" class="fixed top-4 left-1/2 transform -translate-x-1/2 bg-red-500 text-white p-4 rounded-lg mb-4 z-50">
        {{ notification }}
      </div>
      <!-- Trips List -->
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
            <p class="text-sm text-gray-500 mb-2">Price: N{{ trip.price }}</p>
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
      },
      notification: '' // Add a notification data property
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
        Bus: 'https://ticket-simply-africa-cms.s3.af-south-1.amazonaws.com/uploads/bcc/cms/asset/avatar/67765/gallery_g5.jpg',
        'Luxurious-Bus': 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQut1HjoCpMwT3UUMR_-RuYPwt2OuEARI5bNQ&s',
        Travel: '/images/travel.jpg'
      };
      return vehicleImages[vehicleType] || '/images/default.jpg';
    },
    toggleReturnDate(trip) {
      trip.showReturnDate = trip.roundTrip;
      trip.price = trip.roundTrip ? trip.price * 2 : trip.price / 2;
      if (!trip.roundTrip) {
        trip.returnDate = null; // Reset return date if roundTrip is unchecked
      }
    },
    formatReturnDate(trip) {
      if (trip.returnDate) {
        const date = new Date(trip.returnDate);
        trip.returnDate = date.toISOString().split('T')[0]; // Format to yyyy-MM-dd
      }
    },
    async bookTrip(trip) {
      const user_id = localStorage.getItem('user_id'); // Fetch user_id from local storage

      if (!user_id) {
        this.notification = 'Please log in to book a trip.';
        return; // Exit early if user is not logged in
      }
      
      try {
        if (trip.seatsLeft <= 0) {
          this.notification = 'No seats available for this trip.';
          return; // Exit early if no seats available
        }
        
        const response = await axios.post('https://busbooking-eyow.onrender.com/api/v1/booking/book-trip', {
          trip_id: trip.trip_id,
          user_id: user_id,
          isRoundTrip: trip.roundTrip,
          returnTrip: trip.roundTrip ? {
            destination: trip.destination,
            departure: trip.departure,
            time: trip.time,
            price: trip.price / 2, // Assuming round trip price is twice the single trip price
            returnDate: trip.returnDate
          } : null
        });

        if (response.data.success) {
          const { authorization_url, reference } = response.data.data.payment;
          window.location.href = authorization_url;

          // Save reference for later verification
          localStorage.setItem('paymentReference', reference);
          localStorage.setItem('trip_id', trip.trip_id);
          localStorage.setItem('user_id', user_id);
          localStorage.setItem('isRoundTrip', trip.roundTrip);
          if (trip.roundTrip) {
            localStorage.setItem('returnTrip', JSON.stringify({
              destination: trip.destination,
              departure: trip.departure,
              time: trip.time,
              price: trip.price / 2, // Store the return trip price as half of the total
              returnDate: trip.returnDate
            }));
          }
        } else {
          this.notification = 'Failed to initiate booking.';
        }
      } catch (error) {
        console.error('Error booking trip:', error);
        this.notification = 'Error booking trip. Please try again later.';
      }
    },
    
    async fetchTrips() {
      try {
        const response = await axios.get('https://busbooking-eyow.onrender.com/api/v1/booking/alltrip');
        if (response.data.success) {
          this.trips = response.data.data;
        } else {
          this.notification = 'Failed to fetch trips.';
        }
      } catch (error) {
        console.error('Error fetching trips:', error);
        this.notification = 'Error fetching trips. Please try again later.';
      }
    }
  },
  mounted() {
    this.user.user_id = localStorage.getItem('user_id'); // Initialize user_id from local storage
    this.fetchTrips();
    // Check if returning from payment gateway
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






