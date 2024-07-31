<template>
  <section class="fixed top-0 left-0 right-0 bottom-0 flex items-center w-full h-full p-4 bg-coolGray-800 bg-opacity-80 overflow-y-auto">
    <div class="max-w-lg w-full m-auto p-8 bg-white rounded-md">
      <div class="flex items-center justify-center w-10 h-10 mb-4 bg-green-100 rounded-md">
        <!-- Success Icon -->
        <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M12.4733 4.80667C12.4114 4.74418 12.3376 4.69458 12.2564 4.66074C12.1751 4.62689 12.088 4.60947 12 4.60947C11.912 4.60947 11.8249 4.62689 11.7436 4.66074C11.6624 4.69458 11.5886 4.74418 11.5267 4.80667L6.56 9.78L4.47333 7.68667C4.40899 7.62451 4.33302 7.57563 4.24979 7.54283C4.16655 7.51003 4.07767 7.49394 3.98821 7.49549C3.89876 7.49703 3.81049 7.51619 3.72844 7.55185C3.64638 7.58751 3.57216 7.63898 3.51 7.70333C3.44784 7.76768 3.39896 7.84364 3.36616 7.92688C3.33336 8.01011 3.31727 8.099 3.31882 8.18845C3.32037 8.2779 3.33952 8.36618 3.37518 8.44823C3.41084 8.53028 3.46232 8.60451 3.52667 8.66667L6.08667 11.2267C6.14864 11.2892 6.22238 11.3387 6.30362 11.3726C6.38485 11.4064 6.47199 11.4239 6.56 11.4239C6.64801 11.4239 6.73514 11.4064 6.81638 11.3726C6.89762 11.3387 6.97136 11.2892 7.03333 11.2267L12.4733 5.78667C12.541 5.72424 12.595 5.64847 12.6319 5.56414C12.6689 5.4798 12.688 5.38873 12.688 5.29667C12.688 5.2046 12.6689 5.11353 12.6319 5.02919C12.595 4.94486 12.541 4.86909 12.4733 4.80667Z" fill="#2AD168"></path>
        </svg>
      </div>
      <h3 class="mb-2 text-2xl font-semibold text-coolGray-900">Trip Booked Successfully</h3>
      <div class="flex flex-wrap justify-end -m-2">
        <div class="w-full md:w-1/2 p-2">
          <!-- Optional content here -->
        </div>
        <div class="w-full md:w-1/2 p-2">
          <button 
            @click="confirmBooking" 
            class="flex flex-wrap justify-center w-full px-4 py-2.5 bg-green-500 hover:bg-green-600 font-medium text-base text-white border border-green-500 rounded-md shadow-button">
            <p>Confirm</p>
          </button>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  data() {
    return {
      payload: {
        reference: '',
        trip_id: '',
        user_id: '',
        isRoundTrip: false, // Default to false if not provided
        returnDate: '' // Not used in backend but included for completeness
      }
    };
  },
  mounted() {
    this.loadPayload();
  },
  methods: {
    loadPayload() {
      // Retrieve and parse data from localStorage
      const storedReference = localStorage.getItem('paymentReference');
      const storedTripId = localStorage.getItem('trip_id');
      const storedUserId = localStorage.getItem('user_id');
      const storedIsRoundTrip = JSON.parse(localStorage.getItem('isRoundTrip') || 'false');
      const storedReturnDate = localStorage.getItem('returnDate');

      if (storedReference && storedTripId && storedUserId) {
        this.payload = {
          reference: storedReference,
          trip_id: storedTripId,
          user_id: storedUserId,
          isRoundTrip: storedIsRoundTrip,
          returnDate: storedReturnDate || ''
        };
      } else {
        console.warn('One or more pieces of payload data are missing from localStorage');
      }
    },
    async confirmBooking() {
      if (!this.payload.reference || !this.payload.trip_id || !this.payload.user_id) {
        console.error('Incomplete payload data');
        return;
      }

      try {
        const response = await fetch('https://busbooking-eyow.onrender.com/api/v1/booking/verify-booking', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(this.payload),
        });

        const result = await response.json();
        if (response.ok) {
          console.log('Booking confirmation response:', result);
          // Redirect to /trip-history
          this.$router.push('/triphistory');
        } else {
          console.error('Booking failed:', result.error);
        }
      } catch (error) {
        console.error('Error confirming booking:', error);
      }
    },
  },
};
</script>

<style scoped>
/* Add custom styles here if needed */
</style>

