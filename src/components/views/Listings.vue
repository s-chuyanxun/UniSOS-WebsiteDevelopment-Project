<template>
    <div>
        <div id="listing-page">
            <div id="filter">
                <br><br><br><br>
                <h1><i class="fa fa-filter" aria-hidden="true"></i> Filter</h1>
                <br>
                <h3>Type</h3>
                <input type="radio" v-on:change="filter()" v-model="type" value="Notes">Notes
                <br>
                <input type="radio" v-on:change="filter()" v-model="type" value="Tutor">Tutor
                <br>
                <input type="radio" v-on:change="filter()" v-model="type" value="All">All
                <br>
                <br>
                <h3>Rating</h3>
                <input type="radio" v-on:change="filter()" v-model="rating" value=3>
                    <img class="filter-inline" src="../../assets/goldstar.png" alt="star">
                    <img class="filter-inline" src="../../assets/goldstar.png" alt="star">
                    <img class="filter-inline" src="../../assets/goldstar.png" alt="star">
                <br>
                <input type="radio" v-on:change="filter()" v-model="rating" value=2>
                    <img class="filter-inline" src="../../assets/goldstar.png" alt="star">
                    <img class="filter-inline" src="../../assets/goldstar.png" alt="star">
                    <img class="filter-inline" src="../../assets/blackstar.png" alt="star">
                <br>
                <input type="radio" v-on:change="filter()" v-model="rating" value=1>
                    <img class="filter-inline" src="../../assets/goldstar.png" alt="star">
                    <img class="filter-inline" src="../../assets/blackstar.png" alt="star">
                    <img class="filter-inline" src="../../assets/blackstar.png" alt="star">
                <br>
                <input type="radio" v-on:change="filter()" v-model="rating" value="All">All
                <br>
                <br>
            </div>

            <div id="display">
                <div id="searchdiv">
                    <input type="text" id="searchInput" v-on:keyup="filter()" placeholder=" Search by module code...">
                </div>
                <div v-if="this.listingFiltered.length === 0" class="listingview" >
                    <br><h2 id="notfound">No Listings Found!</h2>
                </div>
                <div v-else class="listingview">
                <ul>
                    <li class="listing" v-for="item in listingFiltered" v-bind:key="item.id" v-on:click="item.show = !item.show">
                        <div id="firstpart">
                            <h1>{{item.typeOfList}}</h1>
                            <div v-if="item.profilepic">
                                <div class="image-cropper">
                                    <img :src="item.profilepic" alt="profilepic" class="profile-pic">
                                </div>
                            </div>
                            <div v-else>
                                <div class="image-cropper">
                                    <img src="../../assets/defaultpfp.jpg" alt="profilepic" class="profile-pic">
                                </div>
                            </div>
                            <br>
                            <button class="profile-button" v-bind:id="item.userId" v-on:click="toProfile($event)">{{item.name}}</button>
                            <div v-if="item.rating==0 || item.rating==null">
                                <p>No rating yet</p>
                            </div>
                            <br><br>
                            <div v-if="0<item.rating&&item.rating<=1">
                                <img class="inline" src="../../assets/goldstar.png" alt="star">
                                <img class="inline" src="../../assets/blackstar.png" alt="star">
                                <img class="inline" src="../../assets/blackstar.png" alt="star">
                            </div>
                            <div v-if="1<item.rating&&item.rating<=2">
                                <img class="inline" src="../../assets/goldstar.png" alt="star">
                                <img class="inline" src="../../assets/goldstar.png" alt="star">
                                <img class="inline" src="../../assets/blackstar.png" alt="star">
                            </div>
                            <div v-if="2<item.rating&&item.rating<=3">
                                <img class="inline" src="../../assets/goldstar.png" alt="star">
                                <img class="inline" src="../../assets/goldstar.png" alt="star">
                                <img class="inline" src="../../assets/goldstar.png" alt="star">
                            </div>
                            <br><br>
                        </div>
                        <div id="secondpart">
                            <h2>{{item.module}}</h2>
                            <p>Took in: {{item.took_in}}</p>
                            <p>Final grade: {{item.grade}}</p>
                            <p v-if="item.typeOfList=='Notes'"> Price: ${{item.price}}<p>
                            <p id="addInfo">{{item.addInfo}}</p>
                        </div>
                        <div id="thirdpart">
                            <div v-if="item.userId != currentUser">
                                <button class="chat-button" v-bind:id="item.id" v-if="item.typeOfList=='Notes'" v-on:click="buy(item)">Buy</button>
                                <button class="chat-button" v-bind:id="item.id" v-on:click="toChat($event)">Chat</button>
                            </div>
                            <div v-if="item.userId == currentUser">
                                <button class="chat-button" v-bind:id="item.userId" v-on:click="advertise(item)">Advertise</button>
                                <a v-bind:id="item.id" v-on:click="deleteListing(item)" > 
                                    <i class="fa fa-trash" aria-hidden="true" title="Delete listing?"></i> 
                                </a>
                            </div>
                        </div>
                    </li>
                </ul>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { db } from '../../firebase'
import { auth } from '../../firebase'

export default {
    data() {
        return{
            currentUser: auth.currentUser.uid,
            type:'',
            rating:'',
            listing:[],
            listingFiltered:[],
            wasFiltered:false,
            wasSearched:false
        }
    },
    methods: {
        fetchItems:function() {
            db.collection('listing').get().then((querySnapShot) => {
                querySnapShot.forEach(doc => {
                    var listingData = doc.data();
                    db.collection('users').doc(listingData.userId).get().then(snapshot => {
                        var userData = snapshot.data();
                        listingData.name = userData.name;
                        listingData.email = userData.email;
                        listingData.university = userData.university;
                        listingData.profilepic = userData.profilepic;
                        listingData.bio = userData.bio;
                        listingData.id = doc.id;
                        this.listing.push(listingData);
                    },err => {
                        alert(err.message)
                    });
                },err => {
                    alert(err.message)
                });
            },err => {
                alert(err.message)
            });
            this.listingFiltered = this.listing;
        },
        deleteListing: function(item) {
            var result = confirm("Are you sure you want to delete listing?");
            if (result) {
                db.collection('listing').doc(item.id).delete().then(() => location.reload())
            }
        },
        filter:function() {
            this.wasFiltered = true
            this.listingFiltered = []
            var input = document.getElementById('searchInput');
            var filter = input.value.toUpperCase(); //input of searchbar
            for (var list of this.listing) {
                //assigning rating value
                var assignedRating = 0
                if (list.rating==0 || !(list.rating)) {
                    assignedRating = 0
                } else if (0<list.rating&&list.rating<=1) {
                    assignedRating = 1
                } else if (1<list.rating&&list.rating<=2) {
                    assignedRating = 2
                } else {
                    assignedRating = 3
                }
                var txtValue = list.module
                if ((this.type == list.typeOfList || this.type == '') && (this.rating == assignedRating || this.rating == '')) {
                    if (txtValue.toUpperCase().indexOf(filter) > -1) {
                        this.listingFiltered.push(list);
                    }
                    
                } else if ((this.type == list.typeOfList || this.type == 'All') && (this.rating == assignedRating || this.rating == '')) {
                    if (txtValue.toUpperCase().indexOf(filter) > -1) {
                        this.listingFiltered.push(list);
                    }
                } else if ((this.type == list.typeOfList || this.type == '') && (this.rating == assignedRating || this.rating == 'All')) {
                    if (txtValue.toUpperCase().indexOf(filter) > -1) {
                        this.listingFiltered.push(list);
                    }
                } else if ((this.type == list.typeOfList || this.type == 'All') && (this.rating == assignedRating || this.rating == 'All')) {
                    if (txtValue.toUpperCase().indexOf(filter) > -1) {
                        this.listingFiltered.push(list);
                    }
                }
            } 
        },
        toProfile: function(event) {
            let uid = event.target.getAttribute("id");
            this.$router.push({ name:'profile', params:{ uid:uid } });
        },
        toChat: function(event) {
            let id = event.target.getAttribute("id");
            this.$router.push({ name:'chat', params:{ id:id } });
        },
        buy: function(item) {
            db.collection('users').doc(auth.currentUser.uid).get().then(
                snapshot => {
                    var myNotes = snapshot.data().myNotes;
                    if (myNotes == null) {
                        myNotes = {}
                    }
                    var id = item.id;
                    if (id in myNotes) {
                        alert("Already Bought!")
                    } else {
                        var cfm = confirm("Buy notes for $" + item.price + "?");
                        if (cfm) {
                            this.$router.push({ name:'payment', params:{itemid:item.id,notes:true} });
                        }
                    }
                }
            );
        },
        advertise: function(item) {
            db.collection("listing").doc(item.id).get().then(
                snapshot => {
                    var data = snapshot.data();
                    if (data.advertise != null) {
                        alert("Already advertised!")
                    } else {
                        var adv = confirm("Advertise for $5?")
                        if (adv) {
                            this.$router.push({ name:'payment', params:{itemid:item.id,notes:false} });
                        }
                    }
                }
            )
        },
    },
    created() {
        this.fetchItems();
    }
}
</script>


<style scoped>
@font-face {
    font-family: 'FredokaOne';
    src: url('/fonts/fredokaone-regular-webfont.woff2') format('woff2'),
         url('/fonts/fredokaone-regular-webfont.woff') format('woff');
    font-weight: normal;
    font-style: normal;
}
#listing-page {
    background:  #47E4E4;
    display: flex;
    padding: 20px 30px 20px 30px;
    gap: 30px;
    min-width: 940px;
}
#filter {
    flex: 1;
    font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
    font-size: 18px;
}
#display {
    flex: 9;
    font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
}
.listingview {
    padding: 0px 40px 20px 0px;
    background-color: whitesmoke;
    border-radius: 25px;
    box-shadow: inset 0 0 10px #000000;
    margin: 12px;
}
.listing {
    display: flex;
    align-items: center;
    border-bottom: black solid;
    list-style-type: none;
}
.listing:last-child {
    border-bottom: none;
}
#firstpart{
    flex: 2;
    font-family: 'FredokaOne';
    padding: 0px 30px;
}
.image-cropper {
  height: 120px;
  width: 120px;
  overflow: hidden;
  border-radius: 50%;
}
.profile-pic {
  object-fit: cover;
  height: 100%;
  width: 100%;
}
.profile-button {
    font-family: 'FredokaOne';
    border-radius: 8px;
    cursor: pointer;
    font-size: 16px;
    color: white;
    border: none;
    padding: 5px;
    background-color: #25abb4;
    cursor: pointer;
    text-align: center;
}
.profile-button:hover {
    background-color: black;
}
#secondpart{
    flex: 6;
}
#addInfo{
    white-space:pre-line;
}
#thirdpart{
    flex: 2;
    text-align: center;
}
#thirdpart a{
    font-size: 26px;
    cursor: pointer;
    padding: 10px;
    color: rgb(87, 85, 85);
}
#thirdpart a:hover {
    color: rgba(250, 24, 24, 0.904);
    transition-duration: 0.4s;
    outline: 2px;
    outline-color: black;
}
.chat-button {
    border-radius: 8px;
    border: none;
    padding: 10px 20px;
    background-color:darkgrey;
    color: white;
    font-family: 'FredokaOne';
    cursor: pointer;
    transition-duration: 0.4s;
    font-size: 20px;
    margin:5px;
}
.chat-button:hover {
    background-color: black;
}
.inline {
  display: inline-block;
  vertical-align: middle;
  height:27px;
  width:27px;
}
.filter-inline {
  display: inline-block;
  vertical-align: middle;
  height:18px;
  width:18px;
}
#searchdiv {
    padding: 0px 10px;
}
#searchInput {
    background-image: url('../../assets/search.png');
    background-position: 10px 10px;
    background-repeat: no-repeat;
    background-size: 25px;
    width: 100%;
    font-size: 18px; /* Increase font-size */
    padding: 12px 20px 12px 40px; /* Add some padding */
    border: 1px solid #ddd; /* Add a grey border */
    margin: 12px 0px; /* Add some space below the input */
    font-family: 'FredokaOne';
    border-radius: 25px;
    box-sizing: border-box;
}
#notfound {
    font-family: 'FredokaOne';
    text-align: center;
    color: rgb(100, 100, 100);
}
</style>