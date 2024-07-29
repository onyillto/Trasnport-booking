<template>
  <div class="trip-list">
    <div v-if="trips.length === 0" class="no-trips">No trips available</div>
    <div v-for="trip in trips" :key="trip._id" class="trip-card">
      <div class="trip-details">
        <div class="destination text-green-600 font-bold"><span class="">Destination</span>: {{ trip.destination }}</div>
        <div class="departure font-bold"><span class="font-bold">Departure</span>: {{ trip.departure }}</div>
        <div class="time">{{ formatDateTime(trip.returnDate, trip.time) }}</div>
        <div class="price">₦{{ trip.price }}</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      trips: [] // Initialize as an empty array
    };
  },
methods: {
  formatDateTime(date, time) {
    const dateTimeString = `${date}T${time}`;
    const dateTime = new Date(dateTimeString);
    
    if (isNaN(dateTime.getTime())) {
      return "Invalid Date";
    }
    
    return dateTime.toLocaleString("en-NG", {
      year: "numeric",
      month: "long",
      day: "numeric",
      hour: "numeric",
      minute: "numeric",
      hour12: true
    });
  },
  loadTripsFromLocalStorage() {
    const returnTrip = localStorage.getItem('returnTrip');
    const price = localStorage.getItem('price'); // Retrieve the updated price

    if (returnTrip) {
      try {
        const trip = JSON.parse(returnTrip);

        // Use price from returnTrip if available, otherwise use the price from local storage
        trip.price = trip.price || (price ? parseFloat(price) : trip.price);

        this.trips = [trip]; // Wrap in an array to match the expected structure
      } catch (error) {
        console.error("Error parsing return trip data from local storage:", error);
        this.trips = [];
      }
    } else {
      this.trips = [];
    }
  }
}

,
  mounted() {
    // Load trips from local storage when component is mounted
    this.loadTripsFromLocalStorage();
  }
};
</script>

<style scoped>
.trip-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
  padding: 20px;
}

.trip-card {
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
}

.trip-card:hover {
  transform: translateY(-5px);
}

.trip-details {
  padding: 16px;
}

.destination, .departure, .time, .price {
  margin-bottom: 8px;
  font-size: 1.1rem;
}

.time {
  color: #666;
}

.price {
  font-weight: bold;
  font-size: 1.2rem;
  color: #008cba;
}

.no-trips {
  text-align: center;
  font-size: 1.2rem;
  color: #888;
}
</style>

