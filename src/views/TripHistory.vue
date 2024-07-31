<template>
  <div class="trip-list">
    <div v-if="trips.length === 0" class="no-trips">No trips available</div>
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
      // Parse the ISO 8601 date string
      const dateTime = new Date(date);
      
      if (isNaN(dateTime.getTime())) {
        // Return a fallback string if the date is invalid
        return "Invalid Date";
      }

      // Append time to the parsed date
      const [hours, minutes, period] = time.split(/[:\s]/);
      dateTime.setHours(period === 'PM' ? parseInt(hours) + 12 : parseInt(hours), parseInt(minutes));
      
      // Format the date and time using toLocaleString
      return dateTime.toLocaleString("en-NG", {
        year: "numeric",
        month: "long",
        day: "numeric",
        hour: "numeric",
        minute: "numeric",
        hour12: true
      });
    },
    async loadTrips() {
      const userId = localStorage.getItem('user_id'); // Retrieve user_id from local storage
      if (!userId) {
        console.error("User ID not found in local storage.");
        return;
      }

      try {
        console.log('Fetching trips for user_id:', userId); // Debugging log

        const response = await fetch(`https://busbooking-eyow.onrender.com/api/v1/booking/user-trip?user_id=${encodeURIComponent(userId)}`, {
          method: 'GET', // Correctly using GET method
          headers: {
            'Content-Type': 'application/json',
          }
        });

        if (!response.ok) {
          throw new Error("Failed to fetch trips.");
        }

        const data = await response.json();
        console.log('Fetched data:', data); // Debugging log

        if (data.success) {
          this.trips = data.data || [];
        } else {
          console.error("Failed to fetch trips: ", data.message);
          this.trips = []; // Ensure trips is set to an empty array in case of error
        }
      } catch (error) {
        console.error("Error fetching trips:", error);
        this.trips = []; // Ensure trips is set to an empty array in case of error
      }
    }
  },
  mounted() {
    // Load trips when component is mounted
    this.loadTrips();
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
</style>



