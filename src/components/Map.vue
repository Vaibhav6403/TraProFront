<template>
  <div>
    <div>
      <div class="navbar">
        <div class="navbar-pri">
          <ul>
            <template v-for="(filter, index) in filters" :key="index">
              <li @click="filterLocations(filter)">{{ filter }}</li>
            </template>
            <div class="form-check form-switch">
              <input
                class="form-check-input"
                type="checkbox"
                role="switch"
                id="flexSwitchCheckDefault"
                @change="socialModeChange"
                v-model="socialmode"
              />
              <label class="form-check-label" for="flexSwitchCheckDefault"
                >Social Mode</label
              >
            </div>
            <button @click="logOut">LogOut</button>
            <div class="user-search">
              <input
                type="text"
                v-model="searchQuery"
                placeholder="Search users..."
              />
              <button @click="searchUsers">Search</button>

              <ul class="list-group position-absolute w-100 z-3 mt-1">
                <li
                  v-for="user in searchResults"
                  :key="user.id"
                  class="list-group-item d-flex justify-content-between align-items-center"
                >
                  <span>{{ user.username }}</span>
                  <!-- <span>{{ userFriends.value && !userFriends.value.includes(user.username) }}</span> -->

                  <button
                    v-if="isFriendCheck(user.username)"
                    class="btn btn-sm btn-primary"
                    @click="addFriend(user.username)"
                  >
                    Add Friend
                  </button>

                  <!-- <span
                    v-else
                    @click="removeFriend(user.id)"
                    style="cursor: pointer"
                  >
                    ❌
                  </span> -->
                </li>
              </ul>
            </div>
            <div>
              <div class="notification-bell">
                <i class="fa-solid fa-bell"></i>
                <span
                  v-if="friendRequestsCheck"
                  class="notification-dot"
                ></span>
              </div>
              <ul class="list-group position-absolute w-100 z-3 mt-1">
                <li
                  v-for="(friend, index) in friendRequests"
                  :key="index"
                  class="list-group-item d-flex justify-content-between align-items-center"
                >
                  <span>{{ friend }}</span>
                  <!-- <span>{{ userFriends.value && !userFriends.value.includes(user.username) }}</span> -->

                  <button
                    class="btn btn-sm btn-primary"
                    @click="acceptFriendRequest(friend)"
                  >
                    Accept Request
                  </button>

                </li>
              </ul>
            </div>
            <button class="btn btn-success" @click="showCreateTripModal">
              Create Trip
            </button>
          </ul>
        </div>
        <div class="navbar-sec"></div>
      </div>
    </div>
    <div class="main-container d-flex">
      <div class="sidebar" :class="{ collapsed: isSidebarCollapsed }">
        <div class="trip-div" v-if="!isSidebarCollapsed">
          <ul>
            <li
              v-for="(trip, index) in trips"
              :key="index"
              @click="openTripChat(trip)"
            >
              {{ trip.name }}
            </li>
          </ul>
        </div>
        <div class="arrow-div" @click="toggleSidebar">
          <i class="fa-solid fa-arrow-right"></i>
        </div>
        <div
          class="arrow-div"
          @click="toggleSidebar"
          v-if="!isSidebarCollapsed"
        >
          <i class="fa-solid fa-arrow-left"></i>
        </div>
      </div>
      <div class="main-content-container">
        <div
          v-if="selectedLocation"
          class="custom-popover"
          :style="{
            top: popoverPosition.y + 'px',
            left: popoverPosition.x + 'px',
          }"
        >
          <h3>{{ selectedLocation.name }}</h3>
          <p>Category: {{ selectedLocation.category }}</p>
          <p>{{ selectedLocation.price }}</p>
          <button @click="selectedLocation = null">Close</button>
        </div>
        <div v-if="!selectedTrip">
          <div id="map"></div>
        </div>
        <div v-else class="chat-container">
          <h5>Chat: {{ selectedTrip.name }}</h5>
          <span>Close</span>
          <div class="chat-box">
            <div
              v-for="(msg, i) in tripMessages[selectedTrip._id]"
              :key="i"
              class="chat-message"
            >
              <strong>{{ msg.senderName }}:</strong> {{ msg.content }}
            </div>
          </div>
          <input
            v-model="chatInput"
            @keyup.enter="sendMessage"
            placeholder="Type a message..."
          />
        </div>

        <div
          class="modal fade"
          id="exampleModal"
          tabindex="-1"
          aria-labelledby="exampleModalLabel"
          aria-hidden="true"
        >
          <div class="modal-dialog">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" id="exampleModalLabel">Add Location</h5>
                <button
                  type="button"
                  class="btn-close"
                  data-bs-dismiss="modal"
                  aria-label="Close"
                ></button>
              </div>
              <div class="modal-body">
                <div class="form-group">
                  <input
                    type="text"
                    placeholder=" "
                    id="name"
                    v-model="locationData.name"
                  />
                  <label for="username">Name</label>
                </div>
                <div class="form-group">
                  <label for="experienceType" class="select-label"
                    >Experience Type</label
                  >
                  <select
                    id="experienceType"
                    v-model="locationData.experienceType"
                  >
                    <option disabled value="">Select experience</option>
                    <option
                      v-for="option in locationData.experienceOptions"
                      :key="option"
                    >
                      {{ option }}
                    </option>
                  </select>
                </div>

                <!-- Preference -->
                <div class="form-group">
                  <label for="preference" class="select-label"
                    >Preference</label
                  >
                  <select id="preference" v-model="locationData.preference">
                    <option disabled value="">Select preference</option>
                    <option
                      v-for="option in locationData.preferenceOptions"
                      :key="option"
                    >
                      {{ option }}
                    </option>
                  </select>
                </div>

                <!-- Mood-based -->
                <div class="form-group">
                  <label for="moodBased" class="select-label">Mood-based</label>
                  <select id="moodBased" v-model="locationData.moodBased">
                    <option disabled value="">Select mood</option>
                    <option
                      v-for="option in locationData.moodOptions"
                      :key="option"
                    >
                      {{ option }}
                    </option>
                  </select>
                </div>

                <!-- Time of Day -->
                <div class="form-group">
                  <label for="timeOfDay" class="select-label"
                    >Time of Day</label
                  >
                  <select id="timeOfDay" v-model="locationData.timeOfDay">
                    <option disabled value="">Select time</option>
                    <option
                      v-for="option in locationData.timeOfDayOptions"
                      :key="option"
                    >
                      {{ option }}
                    </option>
                  </select>
                </div>

                <div class="form-group">
                  <input
                    type="text"
                    placeholder=" "
                    id="price"
                    v-model="locationData.price"
                  />
                  <label for="price">Price</label>
                </div>
              </div>
              <div class="modal-footer">
                <button
                  type="button"
                  class="btn btn-secondary"
                  data-bs-dismiss="modal"
                >
                  Close
                </button>
                <button
                  type="button"
                  class="btn btn-primary"
                  @click="addLocation()"
                >
                  Add Location
                </button>
              </div>
            </div>
          </div>
        </div>

        <!-- Create Trip Modal -->
        <div
          class="modal fade"
          id="createTripModal"
          tabindex="-1"
          aria-labelledby="createTripLabel"
          aria-hidden="true"
        >
          <div class="modal-dialog modal-lg">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" id="createTripLabel">Create a Trip</h5>
                <button
                  type="button"
                  class="btn-close"
                  data-bs-dismiss="modal"
                  aria-label="Close"
                ></button>
              </div>
              <div class="modal-body">
                <input
                  type="text"
                  class="form-control mb-3"
                  v-model="tripData.name"
                  placeholder="Trip Name"
                />

                <label>Select Members:</label>
                <div
                  class="form-check"
                  v-for="friend in userFriends"
                  :key="friend"
                >
                  <input
                    type="checkbox"
                    class="form-check-input"
                    :value="friend"
                    v-model="tripData.members"
                  />
                  <label class="form-check-label">{{ friend }}</label>
                </div>

                <!-- Placeholder for chat -->
                <div class="mt-3">
                  <label>Group Chat (Coming Soon)</label>
                  <textarea
                    class="form-control"
                    rows="3"
                    disabled
                    placeholder="Chat will be enabled after trip creation..."
                  ></textarea>
                </div>
              </div>

              <div class="modal-footer">
                <button class="btn btn-secondary" data-bs-dismiss="modal">
                  Cancel
                </button>
                <button class="btn btn-primary" @click="createTrip">
                  Create
                </button>
              </div>
            </div>
          </div>
        </div>

        <!-- Social Clock Buttons -->
        <div class="social-clock">
          <div class="social-clock__list" :class="{ open: isMenuOpen }">
            <button
              class="social-clock__button twitter"
              title="Pointer"
              @click="togglePointer('click')"
            >
              <i class="fa-solid fa-arrow-pointer"></i>
            </button>
            <button
              class="social-clock__button github"
              title="Location"
              @click="togglePointer('current')"
            >
              <i class="fa-solid fa-location-crosshairs"></i>
            </button>
          </div>

          <button
            class="social-clock__trigger add-button"
            @click="toggleMenu"
            aria-label="Toggle Menu"
          >
            <!-- <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512">
          <path
            d="M352 224c53 0 96-43 96-96s-43-96-96-96s-96 43-96 96c0 4 .2 8 .7 11.9l-94.1 47C145.4 170.2 121.9 160 96 160c-53 0-96 43-96 96s43 96 96 96c25.9 0 49.4-10.2 66.6-26.9l94.1 47c-.5 3.9-.7 7.8-.7 11.9c0 53 43 96 96 96s96-43 96-96s-43-96-96-96c-25.9 0-49.4 10.2-66.6 26.9l-94.1-47c.5-3.9 .7-7.8 .7-11.9s-.2-8-.7-11.9l94.1-47C302.6 213.8 326.1 224 352 224z"
          ></path>
        </svg> -->
            <i class="fa-solid fa-plus p-2"></i>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, reactive, onUnmounted } from "vue";
import maplibregl from "maplibre-gl";
import markerImage from "../assets/mapmarker.png";
import { Modal } from "bootstrap";
import axios from "axios";
import { useRouter } from "vue-router";
import {
  socket,
  connectSocket,
  subscribeToTrip,
  registerMessageHandler,
  isConnected,
  messages,
  tripMessages
} from "../webSocket";
import Navbar from "./Navbar.vue";
const modalInstance = ref(null);
const map = ref(null);
const isMenuOpen = ref(false);
const locations = ref([]);
const pointerMode = ref("current");
const socialmode = ref(false);
const searchResults = ref([]);
const userFriends = ref([]);
const friendRequests = ref([]);
const searchQuery = ref("");
const router = useRouter();
const coordinatesCurr = reactive({
  lat: 0,
  lon: 0,
});
const locationData = reactive({
  name: "",
  latitude: "",
  longitude: "",
  experienceType: "",
  preference: "",
  moodBased: "",
  timeOfDay: "",
  maxTime: "",
  price: "",
  persons: "",
  username: "",
  experienceOptions: [
    "Must-visit",
    "Scenic",
    "Hidden Gem",
    "Budget Friendly",
    "Lively",
    "Peaceful",
    "Aesthetic",
  ],
  preferenceOptions: ["Family", "Date", "Friends", "Solo", "Group Trips"],
  moodOptions: ["Romantic", "Chill", "Adventurous", "Unique", "Trending"],
  timeOfDayOptions: [
    "Best at Sunrise",
    "Best at Sunset",
    "Night View",
    "Day Activity",
  ],
});
const markers = reactive([]);
const isShowTripModal = ref(false);
const filters = reactive(["hotel", "office", "family", "date", "All"]);
const selectedLocation = ref(null);
const popoverPosition = ref({ x: 0, y: 0 });
const userInfo = reactive({
  username: "",
  token: "",
});
const tripData = reactive({
  name: "",
  members: [],
});
const trips = ref([]);
const isSidebarCollapsed = ref(true);
const selectedTrip = ref(null);
const chatInput = ref('');
const toggleSidebar = () => {
  isSidebarCollapsed.value = !isSidebarCollapsed.value;
  setTimeout(() => {
    map.value.resize();
  }, 300);
};
onMounted(() => {
  getUserLocation();
  let userId = localStorage.getItem("username");
  userInfo.token = localStorage.getItem("token");
  locationData.username = userId;
  getLocations();
  getUserFriends();
  getFriendRequests();
  getTrips();
  connectSocket(userInfo.token);
  setTimeout(() => {
    trips.value.forEach((trip) => {
      subscribeToTrip(trip._id);
    });
  }, 500);
  registerMessageHandler();
});

onUnmounted(() => {
  clearInterval(intervalId);
});
const showCreateTripModal = () => {
  // debugger;
  isShowTripModal.value = !isShowTripModal.value;

  if (isShowTripModal.value) {
    const tripmodal = document.getElementById("createTripModal");
    const tripmodalInstance = new Modal(tripmodal);
    tripmodalInstance.show();
  }
};

async function  openTripChat(trip) {
  debugger
  selectedTrip.value = trip;
  // subscribeToTrip(trip._id); 
  const request = {
      tripId: trip._id,
    };
    let response = await axios.post(
      "http://localhost:5002/api/user/get-trip-messages",
      request,
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    debugger
  // Ensure message array exists
  if (!tripMessages.value[trip._id]) {
    tripMessages.value[trip._id] = response.data;
  }
}
function sendMessage() {
  if (!chatInput.value || !selectedTrip.value) return;

  const msg = {
    type: 'chat',
    tripId: selectedTrip.value._id,
    content: chatInput.value,
  };

  socket.value.send(JSON.stringify(msg));
  chatInput.value = '';
}
const createTrip = async () => {
  // debugger;
  try {
    tripData.members.push(locationData.username);
    const request = {
      createdBy: locationData.username,
      name: tripData.name,
      members: tripData.members,
    };
    let response = await axios.post(
      "http://localhost:5002/api/user/create-trip",
      request,
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
  } catch (error) {
    console.error("the error in creating trip is", error);
  }
};
const getTrips = async () => {
  try {
    const request = {
      username: locationData.username,
    };
    let response = await axios.post(
      "http://localhost:5002/api/user/get-trips",
      request,
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    console.log(response);
    trips.value = response.data;
  } catch (error) {
    console.error("the error in creating trip is", error);
  }
};
const acceptFriendRequest = async (friendUsername) => {
  try {
    let request = {
      username: locationData.username,
      friendUsername: friendUsername,
    };
    let response = await axios.post(
      "http://localhost:5002/api/user/accept-friend-request",
      request,
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    // debugger;
    if (response.status === 200) {
      friendRequests.value = friendRequests.value.filter(
        (username) => username !== friendUsername
      );
    }
    console.log(response);
  } catch (error) {
    console.error("the error in accept friend request is", error);
  }
};

const getFriendRequests = async () => {
  try {
    let request = {
      username: locationData.username,
    };
    let response = await axios.post(
      "http://localhost:5002/api/user/get-friend-requests",
      request,
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    friendRequests.value = response.data.friendRequestUsernames;
  } catch (error) {
    console.error("the error in the getting friend request is", error);
  }
};
const intervalId = setInterval(getFriendRequests, 10000);

const getUserFriends = async () => {
  try {
    let request = {
      username: locationData.username,
    };
    let response = await axios.post(
      "http://localhost:5002/api/user/get-friends",
      request,
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    userFriends.value = response.data.userFriends.map(
      (friend) => friend.username
    );
  } catch (error) {
    console.error("the error in the get user friends is", error);
  }
};
const isFriendCheck = (username) => {
  // debugger;
  return !userFriends.value.includes(username);
};
const friendRequestsCheck = () => {
  return friendRequests && friendRequests.length;
};
const addFriend = async (friendUsername) => {
  try {
    let request = {
      username: locationData.username,
      friendUsername: friendUsername,
    };
    let response = await axios.post(
      "http://localhost:5002/api/user/add-friend",
      request,
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    console.log("user added", response);
  } catch (error) {
    console.error("the error in add friends is", error);
  }
};
const searchUsers = async () => {
  try {
    let request = {
      username: searchQuery.value,
    };
    let response = await axios.post(
      "http://localhost:5002/api/user/search-friends",
      request,
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    // debugger;
    searchResults.value = response.data.friends;
    // console.log(response.data.friends);
    // searchResults.length = 0; // clear it first
    // response.data.friends.forEach((friend) => {
    //   searchResults.push(friend)}
    // );
    console.log(searchResults.value.length);
  } catch (error) {}
};

const getLocations = async () => {
  try {
    let request = {
      username: locationData.username,
    };
    const response = await axios.post(
      "http://localhost:5002/api/user/get-locations",
      request,
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    locations.value = response.data.locations;
    createMarkers(locations.value);

    console.log(response);
  } catch (error) {
    console.error("the error occured is", error);
  }
};

const createMarkers = (locations) => {
  // debugger;
  locations.forEach((location) => {
    let locationCor = location.location.coordinates;
    const markerEl = document.createElement("img");
    markerEl.src = markerImage;
    markerEl.style.width = "30px";
    markerEl.style.height = "30px";
    markerEl.dataset.id = location._id;
    let marker = new maplibregl.Marker({ element: markerEl })
      .setLngLat(locationCor)
      .addTo(map.value);
    marker.category = location.category;
    markerEl.addEventListener("click", () => {
      selectedLocation.value = location;

      const [lng, lat] = location.location.coordinates;
      const point = map.value.project([lng, lat]);

      popoverPosition.value = { x: point.x, y: point.y };
    });
    markers.push(marker);
  });
};

const filterLocations = (location) => {
  markers.forEach((marker) => {
    const markerCategory = marker.category;
    const isVisible = markerCategory == location || location === "All";
    marker.getElement().style.display = isVisible ? "block" : "none";
  });
};

function getUserLocation() {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      (position) => {
        coordinatesCurr.lat = position.coords.latitude;
        coordinatesCurr.lon = position.coords.longitude;
        initMap([coordinatesCurr.lon, coordinatesCurr.lat]);
      },
      (error) => {
        console.warn("Geolocation error:", error.message);
        initMap([-0.1278, 51.5074]); // Fallback to London
      },
      {
        enableHighAccuracy: true,
        timeout: 10000,
        maximumAge: 0,
      }
    );
  } else {
    alert("Geolocation is not supported by your browser.");
    initMap([-0.1278, 51.5074]); // Fallback to London
  }
}

function initMap(centerCoords) {
  if (map.value) {
    map.value.remove();
  }

  map.value = new maplibregl.Map({
    container: "map",
    style: {
      version: 8,
      sources: {
        osm: {
          type: "raster",
          tiles: ["https://a.tile.openstreetmap.org/{z}/{x}/{y}.png"],
          tileSize: 256,
          attribution: "&copy; OpenStreetMap contributors",
        },
      },
      layers: [
        {
          id: "osm-layer",
          type: "raster",
          source: "osm",
          minzoom: 0,
          maxzoom: 19,
        },
      ],
    },
    center: centerCoords,
    zoom: 13,
  });
  map.value.on("click", (e) => {
    if (pointerMode.value == "click") {
      const clickedLngLat = e.lngLat;
      console.log("The clicked lat long is", clickedLngLat);
      const modalEl = document.getElementById("exampleModal");
      modalInstance.value = new Modal(modalEl);
      locationData.latitude = clickedLngLat.lat;
      locationData.longitude = clickedLngLat.lng;
      modalInstance.value.show();
      pointerMode.value = "";
    }
  });

  const markerEl = document.createElement("img");
  markerEl.src = markerImage;
  markerEl.style.width = "30px";
  markerEl.style.height = "30px";

  new maplibregl.Marker({ element: markerEl })
    .setLngLat(centerCoords)
    .setPopup(new maplibregl.Popup().setText("You are here"))
    .addTo(map.value);

  //   map.value.on("move", () => {
  //     debugger
  //   if (selectedLocation.value) {
  //     const [lng, lat] = selectedLocation.value.location.coordinates;
  //     const point = map.value.project([lng, lat]);
  //     popoverPosition.value = { x: point.x, y: point.y };
  //   }
  // });
}

function toggleMenu() {
  isMenuOpen.value = !isMenuOpen.value;
}
function togglePointer(value) {
  pointerMode.value = value;
  if (pointerMode.value == "current") {
    const modalEl = document.getElementById("exampleModal");
    modalInstance.value = new Modal(modalEl);
    locationData.latitude = coordinatesCurr.lat;
    locationData.longitude = coordinatesCurr.lon;
    modalInstance.value.show();
  }
}
const addLocation = async () => {
  let request = {
    name: locationData.name,
    latitude: locationData.latitude,
    longitude: locationData.longitude,
    category: locationData.category,
    preference: locationData.preference,
    price: locationData.price,
    persons: locationData.persons,
    username: locationData.username,
    experienceType: locationData.experienceType,
    preference: locationData.preference,
    moodBased: locationData.moodBased,
    timeOfDay: locationData.timeOfDay,
  };
  try {
    let response = await axios.post(
      "http://localhost:5002/api/user/add-location",
      request,
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    if (response.status == 201) {
      console.log("response", response);
      modalInstance.value.hide();
    }
    const markerEl = document.createElement("img");
    markerEl.src = markerImage;
    markerEl.style.width = "30px";
    markerEl.style.height = "30px";

    new maplibregl.Marker({ element: markerEl })
      .setLngLat(response.data.location.coordinates)
      .addTo(map.value);
    map.value.flyTo({ center: response.data.location.coordinates, zoom: 14 });
  } catch (error) {
    console.error(error);
  }
};
const socialModeChange = async () => {
  socialmode.value = !socialmode.value;
  if (socialmode) {
    try {
      let request = {
        username: locationData.username,
      };
      let response = await axios.post(
        "http://localhost:5002/api/user/get-friends-location",
        request,
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );
      createMarkers(response?.data?.locations);
    } catch (error) {}
  } else {
  }
};
const logOut = async () => {
  localStorage.removeItem("token");
  localStorage.removeItem("userId");
  localStorage.removeItem("username");
  router.push("/login");
};
</script>

<style scoped>
#map {
  width: 100;
  height: 100vh;
  border: 1px solid #ccc;
  position: relative;
}

.add-button {
  background-color: #007bff;
  color: white;
  font-size: 2rem;
  border-radius: 50%;
  padding: 10px;
  border: none;
  z-index: 20;
  cursor: pointer;
}

.maplibregl-control-container {
  display: none;
}

/* Social Clock Styles */
.social-clock {
  position: absolute !important;
  top: 90%;
  right: 10%;
  transform: translate(50%, -50%);
  height: 16rem;
  width: 16rem;
  border-radius: 16rem;
  pointer-events: none;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: all 0.5s ease;
  z-index: 15;
}

/* List of buttons */
.social-clock__list {
  height: 4.5rem; /* 4rem button + 0.5rem padding */
  width: 4.5rem;
  left: calc(50% - 2.25rem); /* center horizontally */
  top: calc(50% - 2.25rem); /* center vertically */
  position: absolute;
  pointer-events: none;
  transition: transform 0.3s ease-in-out, opacity 0.3s ease-in-out;
  opacity: 0;
  transform: scale(0);
  display: flex;
  justify-content: center;
  align-items: center;
}

/* When menu open, show buttons */
.social-clock__list.open {
  pointer-events: all;
  opacity: 1;
  transform: scale(1);
}

/* Individual buttons */
.social-clock__button {
  background: none;
  border: none;
  border-radius: 4rem;
  box-sizing: content-box;
  cursor: pointer;
  height: 4rem;
  width: 4rem;
  left: calc(50% - 2rem); /* center horizontally */
  top: calc(50% - 4rem / 2); /* center vertically */
  position: absolute;
  transform-origin: 50% 50%;
  pointer-events: all;
  transition: transform 0.3s ease;
  display: flex;
  justify-content: center;
  align-items: center;
}

.social-clock__button:nth-child(1) {
  transform: rotate(-135deg) translate(5.5rem) rotate(135deg);
}

.social-clock__button:nth-child(2) {
  transform: rotate(-45deg) translate(5.5rem) rotate(45deg);
}
/* Button colors */
.social-clock__button.twitter {
  background-color: rgba(29, 161, 242, 0.3);
  border: 2px solid rgba(29, 161, 242, 0.7);
  transition: background-color 0.3s ease, border-color 0.3s ease;
}

.social-clock__button.twitter:hover {
  background-color: rgba(29, 161, 242, 1);
  border-color: rgba(29, 161, 242, 1);
}

.social-clock__button.github {
  background-color: rgba(0, 0, 0, 0.8);
  border: 2px solid rgba(0, 0, 0, 0.9);
  transition: background-color 0.3s ease, border-color 0.3s ease;
}

.social-clock__button.github:hover {
  background-color: rgba(0, 0, 0, 1);
  border-color: rgba(0, 0, 0, 1);
}

/* Icons inside buttons */
.social-clock__button i {
  color: #fff;
  font-size: 1.8rem;
}

/* Trigger button */
.social-clock__trigger {
  background: #000;
  border: none;
  border-radius: 4rem;
  height: 4rem;
  width: 4rem;
  z-index: 3;
  pointer-events: all;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}

.social-clock__trigger svg {
  fill: #fff;
  height: 60%;
  width: 60%;
}
.form-group {
  position: relative;
  margin-bottom: 20px;
}
.form-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
  transition: border-color 0.3s ease;
}
.form-group label {
  position: absolute;
  left: 12px;
  top: 12px;
  font-size: 14px;
  color: #777;
  transition: top 0.3s ease, font-size 0.3s ease, color 0.3s ease;
}
.select-label {
  position: relative !important;
  left: 0px !important;
  top: 0px !important;
  margin-right: 5px;
}
.form-group input:focus {
  border-color: #007bff;
  outline: none;
}
.form-group input:focus + label,
.form-group input:not(:placeholder-shown) + label {
  top: -6px;
  font-size: 12px;
  color: #007bff;
  background-color: #ffffff;
  padding: 0 4px;
  border-radius: 4px;
  left: 8px;
}
.navbar {
  position: fixed;
  z-index: 500;
  width: 100%;
  background-color: black;
  color: white;
  padding: 15px 20px;
}
.navbar-pri ul {
  list-style: none;
  padding: 0;
  margin: 0;
  top: 0;
  position: fixed;
  display: flex;
  width: 50%;
  justify-content: space-around;
  padding: 2px 5px;
}
.custom-popover {
  position: absolute;
  background: white;
  padding: 12px;
  border-radius: 6px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  z-index: 999;
  transform: translate(-50%, -100%); /* position above the marker */
}
.user-search {
  position: relative;
}

.search-dropdown,
.list-group.position-absolute {
  max-height: 300px;
  overflow-y: auto;
  background: white;
  z-index: 1050;
  border: 1px solid #ccc;
  top: 25px;
}

.notification-bell {
  position: relative;
  display: inline-block;
  font-size: 24px;
  color: #007bff;
}

.notification-dot {
  position: absolute;
  top: 0;
  right: 0;
  width: 10px;
  height: 10px;
  background-color: red;
  border-radius: 50%;
  border: 2px solid white;
}
.main-content-container {
  width: 100%;
  height: 100vh;
  flex: 1;
}
.sidebar {
  position: relative;
  width: 15vw;
  transition: width 0.3s ease;
}
.arrow-div {
  position: absolute;
  top: 200px;
  right: -20px;
  z-index: 200;
  width: 40px;
  height: 40px;
  border-radius: 10px;
  display: flex;
  justify-content: center;
  background-color: #007bff;
  align-items: center;
  cursor: pointer;
  border: 2px solid black;
}
.collapsed {
  width: 2vw !important;
}
.trip-div {
  display: flex;
  flex-direction: column;
  margin-top: 50px;
}
.trip-div ul {
  padding-left: 0px;
}
.trip-div li {
  list-style: none;
  padding: 5px;
  border: 2px solid black;
  margin: 5px;
}
.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 1rem;
  background-color: #f8f9fa;
  border-left: 1px solid #dee2e6;
  margin-top: 30px;
}

.chat-box {
  flex: 1;
  overflow-y: auto;
  border: 1px solid #dee2e6;
  border-radius: 8px;
  padding: 1rem;
  background-color: white;
  margin-bottom: 1rem;
  max-height: 500px; /* or use 100% if parent is constrained */
}

.chat-message {
  margin-bottom: 0.75rem;
  padding: 0.5rem 0.75rem;
  border-radius: 6px;
  background-color: #e9ecef;
  word-wrap: break-word;
}

.chat-message strong {
  color: #343a40;
  margin-right: 0.5rem;
}

.chat-container input {
  border: 1px solid #ced4da;
  padding: 0.5rem 0.75rem;
  border-radius: 6px;
  font-size: 1rem;
  outline: none;
  width: 100%;
}

</style>
