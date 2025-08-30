<template>
  <div class="car-page">
    <!-- Top Navigation -->
    <div class="top-bar">
      <header class="header">
        <h1 @click="goTo('home')">REVVED AUCTION</h1>
      </header>
      <nav class="nav-menu">
        <ul>
          <li><a href="/product">PRODUCTS</a></li>
          <li><a href="/session">AUCTION</a></li>
          <li>CONTACT</li>
          <li><a href="/account-setup">LOGIN</a></li>
        </ul>
      </nav>
    </div>

    <!-- Car Detail Section -->
    <section class="car-detail-section">
      <div v-if="loading" class="loading">Loading car details...</div>

      <div v-else-if="!car || Object.keys(car).length === 0" class="empty-message">
        🚗 Car details not available
      </div>

      <div v-else class="car-container">
        <!-- Left: Car Image -->
        <div class="car-image-section">
          <img :src="car.mediaBase64 || placeholderImage" alt="Car Image" />
          <div class="overlay-360">360° View</div>
        </div>

        <!-- Middle: Vehicle Details -->
        <div class="vehicle-details">
          <h3>Vehicle Details</h3>
          <table>
            <tbody>
              <tr><td>Code</td><td>{{ car.carVIN || 'N/A' }}</td></tr>
              <tr><td>Mileage</td><td>{{ (car.carMileage ?? 0).toLocaleString() }} km</td></tr>
              <tr><td>Key</td><td>{{ car.hasKey ? 'Yes' : 'No' }}</td></tr>
              <tr><td>Status</td><td>{{ car.carStatus || 'Unknown' }}</td></tr>
              <tr><td>Make / Model</td><td>{{ car.carMake }} {{ car.carModel }}</td></tr>
              <tr><td>Year</td><td>{{ car.carYear }}</td></tr>
              <tr><td>Source</td><td>{{ car.source || 'Unknown' }}</td></tr>
            </tbody>
          </table>
        </div>

        <!-- Right: Auction & Actions -->
        <div class="auction-info">
         
          <div class="countdown-bar">
            ⏰ {{ formatCountdown(car.remainingTime) }}
          </div>

          <!-- Current Bids -->
          <div class="bid-info">
            <p>Asking Bid</p>
            <h2>R{{ (car.price ?? 0).toLocaleString() }}</h2>
            <p>Winning Buyer: {{ car.winningBuyer || 'None' }}</p>
          </div>

          <!-- Bid Calculator -->
          <div class="bid-calculator">
            <h4>Bid Calculator</h4>
            <input type="number" v-model.number="userBid" placeholder="Enter your bid" />
            <button @click="calculateBid">Calculate</button>
            <p v-if="calculatedBid">Your bid plus increment: R{{ calculatedBid.toLocaleString() }}</p>
          </div>

          <!-- Login Prompt -->
          <div class="login-prompt">
            <p>Please log in to place your bid.</p>
            <button @click="goTo('login')">Login</button>
            <button @click="goTo('signup')">Sign Up</button>
          </div>

          <!-- Auction Details -->
          <div class="bid-details">
            <p>Bid Increment: R{{ car.bidIncrement ?? 1000 }}</p>
            <p>Winning Bid: R{{ car.currentBid ?? 0 }}</p>
            <p>Subject To Confirmation: Yes</p>
            <p>Expires: {{ formatDate(car.auctionEndTime) }}</p>
            <p>Successful Bids: {{ car.successfulBids ?? 0 }}</p>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import CarService from "@/services/CarService";

export default {
  name: "CarDetailsPage",
  data() {
    return {
      car: {},
      loading: true,
      timer: null,
      placeholderImage: "https://via.placeholder.com/400x300?text=No+Image",
      userBid: null,
      calculatedBid: null,
    };
  },
  methods: {
    goTo(page) {
      this.$router.push({ name: page });
    },
    goBack() {
      this.$router.back();
    },
    async fetchCar() {
      const vin = this.$route.params.vin;
      try {
        const res = await CarService.getCarByVIN(vin);
        const auctionTime = new Date(res.data.auctionEndTime);
        this.car = {
          ...res.data,
          remainingTime: auctionTime - new Date(),
          currentBid: res.data.currentBid ?? 0,
          price: res.data.price ?? 0,
        };
      } catch (err) {
        console.error("Error fetching car:", err);
        this.car = {};
      } finally {
        this.loading = false;
      }
    },
    updateCountdown() {
      if (this.car && this.car.auctionEndTime) {
        this.car.remainingTime = new Date(this.car.auctionEndTime) - new Date();
      }
    },
    formatCountdown(ms) {
      if (!ms || ms <= 0) return "Auction Ended";
      const days = Math.floor(ms / (1000 * 3600 * 24));
      const hours = Math.floor((ms / (1000 * 3600)) % 24);
      const minutes = Math.floor((ms / (1000 * 60)) % 60);
      const seconds = Math.floor((ms / 1000) % 60);
      return `${days}d ${hours}h ${minutes}m ${seconds}s`;
    },
    formatDate(dateStr) {
      if (!dateStr) return "N/A";
      return new Date(dateStr).toLocaleString();
    },
    calculateBid() {
      const increment = this.car.bidIncrement ?? 1000;
      if (this.userBid) {
        this.calculatedBid = this.userBid + increment;
      }
    },
  },
  mounted() {
    this.fetchCar();
    this.timer = setInterval(this.updateCountdown, 1000);
  },
  beforeUnmount() {
    clearInterval(this.timer);
  },
};
</script>

<style scoped>
.car-page {
  font-family: 'Montserrat', sans-serif;
  background: #f8f9fb;
  min-height: 100vh;
  padding: 20px;
}
.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
}
.header h1 {
  cursor: pointer;
  font-weight: 800;
  font-size: 24px;
  color: #2d3436;
}
.nav-menu ul {
  display: flex;
  list-style: none;
  gap: 20px;
}
.nav-menu a {
  text-decoration: none;
  color: #2d3436;
  font-weight: 600;
}
.car-detail-section {
  display: flex;
  justify-content: center;
}
.car-container {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 20px;
  background: #B5B7BB;
  border-radius: 16px;
  box-shadow: 0 6px 20px rgba(0,0,0,0.1);
  padding: 20px;
  max-width: 1200px;
  width: 100%;
}
.car-image-section img {
  width: 100%;
  border-radius: 12px;
  object-fit: cover;
}
.overlay-360 {
  text-align: center;
  background: #BB9999;
  margin-top: 10px;
  padding: 6px;
  border-radius: 6px;
  font-weight: 600;
}
.vehicle-details h3 {
  background: #216D96;
  padding: 10px;
  border-radius: 6px;
  text-align: center;
  margin-bottom: 10px;
}
.vehicle-details table {
  width: 100%;
  border-collapse: collapse;
}
.vehicle-details td {
  padding: 8px;
  border-bottom: 1px solid #ddd;
}
.auction-info .countdown-bar {
  background: #216D96;
  padding: 12px;
  margin-bottom: 15px;
  text-align: center;
  font-weight: bold;
  border-radius: 6px;
  font-size: 16px;
}
.bid-info h2 {
  color: #e17055;
}
.bid-calculator {
  background: #dfe6e9;
  padding: 10px;
  border-radius: 8px;
  margin-bottom: 15px;
}
.bid-calculator input {
  width: 80%;
  padding: 6px;
  margin-right: 5px;
  border-radius: 6px;
  border: 1px solid #b2bec3;
}
.bid-calculator button {
  padding: 6px 10px;
  border: none;
  border-radius: 6px;
  background: #fd79a8;
  color: #fff;
  font-weight: 600;
  cursor: pointer;
}
.login-prompt button {
  margin: 5px 5px 5px 0;
  padding: 10px 15px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
}
.login-prompt button:nth-child(2) {
  background: #55efc4;
}
.login-prompt button:nth-child(3) {
  background: #74b9ff;
}
.bid-details p {
  margin: 5px 0;
}
.loading, .empty-message {
  font-size: 18px;
  text-align: center;
  color: #555;
}
</style>
