<template>
  <div class="trip-list">
    <div v-if="!trips.length" class="no-trips">No trips available</div>
    <div v-else>
      <div class="user-full-name">Full Name: {{ fullName }}</div>
      <div v-for="trip in trips" :key="trip._id" class="trip-card">
        <div class="trip-details">
          <div class="destination text-green-600 font-bold">
            <span>Destination</span>: {{ trip.destination }}
          </div>
          <div class="departure font-bold">
            <span>Departure</span>: {{ trip.departure }}
          </div>
          <div class="time">{{ formatDateTime(trip.bookingDate, trip.time) }}</div>
          <div class="price">₦{{ trip.price }}</div>
          <div class="booking-id font-bold">
            <span>Booking ID</span>: {{ trip.booking_id }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      fullName: '', // Initialize as empty
      trips: [] // Initialize as an empty array
    };
  },
  mounted() {
    // Fetch data when the component is mounted
    this.fetchTrips();
  },
  methods: {
    async fetchTrips() {
      try {
        const response = await fetch('/api/trips'); // Replace with your API endpoint
        const result = await response.json();
        if (result.success) {
          this.fullName = result.data.fullName;
          this.trips = result.data.trips;
        } else {
          console.error('Failed to fetch trips:', result.message);
        }
      } catch (error) {
        console.error('Error fetching trips:', error);
      }
    },
    formatDateTime(date, time) {
      // Parse the ISO 8601 date string and format it
      const dateTime = new Date(date);
      const formattedDate = dateTime.toLocaleDateString(); // Format as needed
      return `${formattedDate} ${time}`;
    }
  }
};
</script>

<style>
/* Add your CSS styling here */
</style>

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

.destination, .departure, .time, .price, .booking-id {
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

.booking-id {
  font-weight: bold;
}

.no-trips {
  text-align: center;
  font-size: 1.2rem;
  color: #888;
}

.user-full-name {
  color: #302525;
  font-size: 1.2rem;
  font-weight: bold;
  margin-bottom: 20px;
}
</style>



