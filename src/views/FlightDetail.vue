<template>
  <div>
    <Navbar />
    <div class="main">
      <b-container class="pt-5">
        <b-row>
          <b-col col lg="8" md="8" sm="12" cols="12" style="width: 100%;">
            <h4 class="text-white mb-4">Contact Person Details</h4>
            <ContactPersonDetail :formBooking="formBooking" :error="error" />
          </b-col>
          <b-col col lg="4" md="4" sm="12" cols="12" style="width: 100%;">
            <h4 class="text-white mb-4">Flight Detail</h4>
            <FlightDetailCard :flight="flight" :total="total" />
          </b-col>
        </b-row>
        <b-row>
          <b-col col lg="8" md="8" sm="12" cols="12" style="width: 100%;">
            <h4 class=" mb-4 mt-4  text-black">
              Passanger Details
            </h4>
            <PassangerDetailTop
              :formPassenger="formPassenger"
              :passenger="passenger"
              :flight="flight"
              :params="params"
              :errorPassenger="errorPassenger"
            />
          </b-col>
        </b-row>
        <b-row>
          <b-col col lg="8" md="8" sm="12" cols="12" style=" width: 100%;">
            <h4 class=" mb-4 mt-5  text-black">
              Passanger Details
            </h4>
            <PassangerDetailBot />
          </b-col>
        </b-row>
        <b-row>
          <b-col col lg="8" md="8" sm="12" cols="12" style=" width: 100%;">
            <div class="text-center">
              <button @click="addBooking" class="btn-payment my-5">
                Procees to Payment
              </button>
            </div>
          </b-col>
        </b-row>
      </b-container>
    </div>
    <Footer />
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import alert from '../mixins/alert'
import Navbar from '../components/Navbar'
import Footer from '../components/Footer'
import ContactPersonDetail from '../components/_base/FlightDetail/ContactPersonDetail'
import FlightDetailCard from '../components/_base/FlightDetail/FlightDetail'
import PassangerDetailTop from '../components/_base/FlightDetail/PassengerDetailTop'
import PassangerDetailBot from '../components/_base/FlightDetail/PassangerDetailBot'

export default {
  name: 'FlightDetail',
  mixins: [alert],
  components: {
    Navbar,
    Footer,
    ContactPersonDetail,
    FlightDetailCard,
    PassangerDetailTop,
    PassangerDetailBot
  },
  data() {
    return {
      formBooking: {
        contactFullName: '',
        contactEmail: '',
        phoneCode: '+62',
        phoneNumber: ''
      },
      formPassenger: [],
      passenger: {
        title: 'Mr.',
        fullName: '',
        nationality: 'Indonesia'
      },
      userId: '',
      error: '',
      errorPassenger: ''
    }
  },
  computed: {
    ...mapGetters({
      setUser: 'setUser',
      params: 'getParams',
      flight: 'getChooseFlight'
    }),
    total() {
      return this.formPassenger.length * this.flight.price
    }
  },
  created() {
    this.userId = this.setUser.userId
  },
  methods: {
    ...mapActions(['postBooking', 'patchFlightCapacity', 'sendNotif']),
    addBooking() {
      if (
        this.formBooking.contactFullName === '' ||
        this.formBooking.contactEmail === '' ||
        this.formBooking.phoneNumber === ''
      ) {
        return (this.error = '*Please fill contact person details ')
      }
      this.error = ''

      if (this.formPassenger.length < 1) {
        return (this.errorPassenger = '*Please add passenger')
      }
      this.errorPassenger = ''

      if (this.params.totalPassanger > 0) {
        if (this.formPassenger.length != this.params.totalPassanger) {
          return (this.errorPassenger = `*Please insert ${this.params
            .totalPassanger -
            this.formPassenger.length} passengers more to book ticket`)
        }
      }
      this.errorPassenger = ''

      const dataBooking = {
        userId: this.userId,
        flightId: this.flight.flightId,
        totalPassenger: this.formPassenger.length,
        totalPayment: this.total,
        contactFullName: this.formBooking.contactFullName,
        contactEmail: this.formBooking.contactEmail,
        contactNumber: this.formBooking.phoneCode + this.formBooking.phoneNumber
      }

      const setData = [dataBooking, ...this.formPassenger]

      console.log(setData)
      const patchFlight = {
        flightId: this.flight.flightId,
        totalPassenger: this.formPassenger.length
      }

      this.patchFlightCapacity(patchFlight)
        .then(result => {
          console.log(result)
          this.postBooking(setData)
            .then(result => {
              this.successAlert(result.data.msg)
              const dataNotif = {
                notifTitle: 'Booking status',
                notifMessage: `Hello there! Your booking status is received and waiting for payment. Booking code: ${result.data.data[0].code} . Pay your ticket here :'${result.data.data[1].midtransUrl}`,
                receiverId: this.setUser.userId
              }
              this.sendNotif(dataNotif)
                .then(result => {
                  this.$toasted.success(result.data.msg)
                })
                .catch(error => {
                  this.errorAlert(error.data.msg)
                })
            })
            .catch(error => {
              this.errorAlert(error.data.msg)
            })
        })
        .catch(err => {
          this.errorAlert(err.data.msg)
        })
    }
  }
}
</script>

<style scoped>
button {
  outline: unset;
}

.main {
  background-image: url('../assets/bg-header.png');
  background-repeat: no-repeat;
  background-size: 100vw;
  background-color: #f5f6fa;
}

.text-white {
  color: #fff;
}

.text-black {
  color: #000;
}

.btn-payment {
  background-color: #2395ff;
  color: #fff;
  font-size: 600;
  padding: 15px 100px;
  border-radius: 10px;
  transition-duration: 0.7s;
  border: unset;
  font-weight: 600;
}

.btn-payment:hover {
  background-color: #037ff2;
  -webkit-box-shadow: 0px 10px 20px rgba(0, 0, 0, 0.2);
  box-shadow: 0px 10px 20px rgba(0, 0, 0, 0.2);
}
</style>
