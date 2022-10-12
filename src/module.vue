<template>
	<private-view title="Timesheet">
		<template #navigation>
        <CustomNavigationLeftSidebar />
        </template>
    <template #sidebar>
      <sidebar-detail-group>
        <sidebar-detail title="Views" icon="layers">
          <v-select
            v-model="view"
            :items="views"
          />
        </sidebar-detail>
      </sidebar-detail-group>
      
			
    </template>
		<template v-if="view =='weekly'">
      <v-card>
        <v-card-title>{{ isMobile ?(new Date(this.dateUnformated).toLocaleDateString('en-us', { weekday:"short",  month:"short", day:"numeric"})):(new Date(this.dateUnformated).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"})) }}</v-card-title>
        <v-card-subtitle>
          <v-btn
            outlined
            class="grey--text mr-4 d-none d-sm-block"
            @click="setToday"
          >
            Today
          </v-btn>
          <v-btn
            fab
            text
            small
            @click="prev"
          >prev
            <v-icon  class="grey--text">
              mdi-chevron-left
            </v-icon>
          </v-btn>
          <v-btn
            fab
            text
            small
            class="ml-2"
            @click="next"
          >next
            <v-icon  class="grey--text">
              mdi-chevron-right
            </v-icon>
          </v-btn>
          <v-spacer></v-spacer>
          
        </v-card-subtitle>
        <v-card-actions>
          <v-button @click="dialog=true">Save</v-button>
        </v-card-actions>
      </v-card>
		<v-tabs v-model="tab" :key="Date.now()">
			<v-tab v-for="item in tabItems" :key="item.id" :style="[item.tab == 'Total' ? 'pointer-events: none;':'']" >{{isMobile && item.tab != 'Total' ?item.tab.substring(0, 1):item.tab}}</v-tab>
		</v-tabs>

		<v-tabs-items v-model="tab" :key="Date.now()">
			<v-tab-item v-for="itemTitle in tabItems" :key="itemTitle.id" >
				<v-divider></v-divider>
          <v-list three-line class="d-flex align-center" v-if="itemTitle.tasks.length == 0"  height="200">
            <v-list-item>
              <v-card style="display:flex;align-items:center;justify-content:center;min-height:200px;text-align:center">
                <v-card-title>
                  "If You Can't Measure It, You Can't Improve It" <br> ....Time
                </v-card-title>
              </v-card>
            </v-list-item>
          
          </v-list>
          
            <v-list class="py-0 teb-contents" v-else  three-line v-for="(item, index) in itemTitle.tasks"  :key="index">
              
              <v-list-item  :key="item.project">
                <v-card>
                  <div>
                    <v-card-title>
                      {{item.project}} &mdash;
                      <span class="text-caption text-sm-subtitle-2 font-weight-regular">{{item.department}}</span>
                    </v-card-title>
                    <v-card-subtitle>{{item.hours}} </v-card-subtitle>
                    <v-card-text>{{item.notes}}</v-card-text>
                  </div>
                  <v-card-actions>
                    <v-button @click="dialogUpdate(item.id)">Edit</v-button>
                  </v-card-actions>
                </v-card>
              </v-list-item>
             
            </v-list>
            
			</v-tab-item>
			
		</v-tabs-items>
    
    <v-drawer v-model="dialog">
	<div>This text will show up in the drawer</div>
</v-drawer>
		</template>
		<template v-else>
			<v-table
				:headers="[
					{
						text: 'Name',
						value: 'name'
					},
					{
						text: 'Phone Number',
						value: 'tel'
					},
					{
						text: 'Contact',
						value: 'person'
					}
				]"
				:items="[
					{
						name: 'Amsterdam',
						tel: '020-1122334',
						person: 'Mariann Rumble'
					},
					{
						name: 'New Haven',
						tel: '(203) 687-9900',
						person: 'Helenka Killely'
					}
				]"
			/>
		</template>
		<!-- <v-button v-on:click="logToConsole">Log collections to console</v-button> -->
	</private-view>
</template>

<script>
import moment from 'moment';
// import { useApi, useStores } from '@directus/extensions-sdk';
export default {
	// setup() {
	// 	const api = useApi();

	// 	const { useCollectionsStore } = useStores();
	// 	const collectionsStore = useCollectionsStore();

	// 	console.log(collectionsStore)
	// },
	data() {
		return {
			collections: null,
			activePage:'',
			dialogm1: '',
    dialog: false,
    absolute: true,
    opacity: 1,
    overlay: false,
    drawer: null,
    tab: [0],
    tabItems: [],
    tabContentItems: [],
    links: [
      'Time',
      'Projects',
      'Team',
      'Reports',
    ],
    projects: [],
    departments: [],
    dateUnformated : new Date().toString(),
    dateShow : new Date().toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"}),
    calendarShow : false,
    task: {
      id: Date.now() + parseInt(Math.random()*100),
      userId:'',
      date: new Date(),
      dateTime: new Date().getTime(),
      project:'',
      department:'',
      hours:'',
      notes:'',
      status:'published'
    },
    date: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),
    menu: false,
    modal: false,
    menu2: false,
    edit: false,
    deleteId:'',
    dialogDelete :false,
    valid:false,
    snackbar: false,
    snackbarText: '',
    start:"",
    end:"",
    team:'',
		views:[
			{
				text: 'Weekly',
				value: 'weekly',
			},
			{
				text: 'Monthly',
				value: 'monthly',
			},
		],
		view : 'weekly',
    isMobile : false,
		
		};
	},
	inject: ['api'],
	mounted() {
		// log the system field so you can see what attributes are available under it
		// remove this line when you're done.
    
    if(window.innerWidth <768){
      this.isMobile = true;
    }
    else{
      this.isMobile = false;
    }
    window.addEventListener('resize', this.handleResize)
		console.log(this.api);

		this.setTabContentItems();
	},
  destroyed() {
    window.removeEventListener('resize', this.handleResize)
  },
	methods: {
		logToConsole: function () {
			console.log(this.collections);
		},
    handleResize (e){
      console.log(e.target.innerWidth);
      if(e.target.innerWidth <768){
        this.isMobile = true;
      }
      else{
        this.isMobile = false;
      }
    },
		setDate (x) {
      // this.dateUnformated = moment(x).format('MMMM Do YYYY, h:mm:ss a');
      // console.log("moment" + this.dateUnformated);
      this.dateUnformated = new Date((new Date(x).getTime()) + (new Date(x).getTimezoneOffset()) * 60000);
      // console.log("date" + this.dateUnformated);
      // this.dateShow = new Date(x).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"});
      this.calendarShow = !this.calendarShow
    },

    setToday () {
      this.dateUnformated = new Date().toString();
      // this.dateShow = new Date().toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"})
    },
    prev () {
      let nextDate = new Date(this.dateUnformated);
      nextDate.setDate(nextDate.getDate() - 1);
      //console.log(nextDate);
      
      this.dateUnformated = nextDate.toString();
      // this.dateShow = nextDate.toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"});
    },
    next () {
      let nextDate = new Date(this.dateUnformated);
      nextDate.setDate(nextDate.getDate() + 1);
      //console.log(nextDate);
      this.dateUnformated = new Date(nextDate).toString();
      // this.dateShow = nextDate.toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"});
    },
    timestrToSec(timestr) {
      // console.log(timestr)
      var parts = timestr.split(":");
      return (parts[0] * 3600) +
            (parts[1] * 60) 
    },
    pad(num) {
      if(num < 10) {
        return "0" + num;
      } else {
        return "" + num;
      }
    },
    formatTime(seconds) {
      // console.log(seconds);
      return [this.pad(Math.floor(seconds/3600)),
              this.pad(Math.floor(seconds/60)%60)
              ].join(":");
    },
    hourCalculator(tabitem){
      // console.log(tabitem);
      for(let i=0;i< this.tabContentItems.length; i++){
        // console.log(this.tabContentItems[i]);
        if(this.tabContentItems[i].day == tabitem.tab || tabitem.tab == 'Total'){
          tabitem.hours = this.formatTime(this.timestrToSec(tabitem.hours) + this.timestrToSec(this.tabContentItems[i].hours));
        }
      }
      this.tabItems[tabitem.id].hours = tabitem.hours;
      return tabitem.hours
    
    },

    
    setTabContentItems(){
      let that = this;
      let currDate = new Date(this.dateUnformated);
      // console.log(currDate.getTime(),moment(currDate).weekday(0).get('date'));
      let startDate = moment(currDate).weekday(0);
      currDate.setDate(startDate.get('date'));
      currDate.setMonth(startDate.get('month'));
      currDate.setFullYear(startDate.get('year'));

      // currDate.setDate(currDate.getDate() - currDate.getDay() +1);
      currDate.setHours("00");
      currDate.setMinutes("00");
      currDate.setSeconds("00");
      this.start = parseFloat(new Date((currDate.getTime()) + currDate.getTimezoneOffset() * 60000).getTime());
      console.log(currDate);
      currDate.setHours("23");
      currDate.setMinutes("59");
      currDate.setSeconds("59");
      currDate.setDate(currDate.getDate() +6);
      console.log(currDate);
      this.end = parseFloat(new Date((currDate.getTime()) + currDate.getTimezoneOffset() * 60000).getTime());
      console.log(this.start,this.end);
			this.api.get(`/items/Timesheet?filter={"_and":[{"dateTime":{"_lte":"${this.end}"}},{"dateTime":{"_gte":"${this.start}"}}]}`).then((res) => {
				this.collections = res.data.data;
				
        if(res.data.data.length > 0) {
          try {
              that.tabContentItems = [];
              for (const doc of res.data.data) {
                console.log(doc);
                that.tabContentItems.push(doc);
              }
              // console.log(that.tabContentItems);
              that.setTabItems();
            } catch (err) {}
        }
        else{
          that.setTabItems();
        }
        console.log(that.tabContentItems);
			});

     


    },
		setTabItems(){
      let curr = new Date(this.dateUnformated);
      let currTime = parseFloat(new Date((curr.getTime()) + curr.getTimezoneOffset() * 60000).getTime());
      // console.log(curr,currTime);
      if(this.start<= currTime && currTime <= this.end){
      let currId = '',
      tabItems = [],
      totalItem = {
        id:"w7"+Math.floor(Math.random() * 1000),
        tab:"Total",
        tasks:[],
        hours:"00:00"
        };
      
      let tabDate = new Date(this.start);
      tabDate.setDate(tabDate.getDate());
      tabDate.setHours("00");
      tabDate.setMinutes("00");
      tabDate.setSeconds("00");

      for (let i = 0; i < 7; i++) {
        let item = {};
        item.id = "w"+i+Math.floor(Math.random() * 1000);
        // console.log(tabDate.getDate());
        let first =(i==0 && tabDate.toLocaleDateString('en-us', { weekday:"long" }) == 'Sunday'? tabDate.getDate(): tabDate.getDate()+1) ;
        console.log(first);
        let day = new Date(tabDate.setDate(first)).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"});
        if((tabDate.getDate()) == (new Date(this.dateUnformated).getDate())){
          currId = i;
          // console.log(currId);
        }
        item.tab = new Date(day).toLocaleDateString('en-us', { weekday:"short"});
        item.date= new Date(day).toString();
        // console.log(i,"-----",item.date)
        item.tasks = [];
        item.hours = "00:00";
        for(let j=0;j<this.tabContentItems.length;j++){
          // console.log(this.tabContentItems[j].date == day);
          if(this.tabContentItems[j].date == day){
            item.tasks.push(this.tabContentItems[j]);
            item.hours = this.formatTime(this.timestrToSec(item.hours) + this.timestrToSec(this.tabContentItems[j].hours));
            totalItem.tasks.push(this.tabContentItems[j]);
            totalItem.hours = this.formatTime(this.timestrToSec(totalItem.hours) + this.timestrToSec(this.tabContentItems[j].hours));
            // console.log(totalItem.hours);
          }
        }
        console.log(item);

       
        tabItems.push(item);
      }
      console.log(totalItem);
      tabItems.push(totalItem);
      this.tabItems = tabItems;
      this.tab = [currId];
      console.log("tab ----- ",this.tabItems,this.tab);
    }
    else{
      // console.log("else part");
      this.setTabContentItems();
    }
    },
    dialogUpdate(id){
      let tasks = this.tabContentItems.filter((tsk) => (tsk.id == id));
      this.task = tasks[0];
      // console.log(tasks);
      this.edit = true;
      this.dialog = true;
    },
    dialogClose(){
      this.setTabContentItems();
      this.dialog = false;
      this.edit = false;
      this.date = new Date((new Date(this.dateUnformated).getTime()) - (new Date(this.dateUnformated).getTimezoneOffset()) * 60000).toISOString().substr(0, 10);
      this.task.id='';
      this.task.date = new Date((new Date(this.date).getTime()) + (new Date(this.date).getTimezoneOffset()) * 60000).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"});
      this.task.dateTime = new Date((new Date(this.date).getTime()) + (new Date(this.date).getTimezoneOffset()) * 60000).getTime();
      this.task.project='';
      this.task.department='';
      this.task.hours='';
      this.task.notes=''
    
      console.log(this.task);
            
    },
    dialogSubmit(){
      let that = this
      if (this.$refs.form.validate()) {
        this.task.id = Date.now() + parseInt(Math.random()*100);
        this.task.userId = this.$auth.user.id;
        this.task.date = String(new Date((new Date(this.date).getTime()) + (new Date(this.date).getTimezoneOffset()) * 60000).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"}));
        this.task.dateTime = parseFloat(new Date((new Date(this.date).getTime()) + (new Date(this.date).getTimezoneOffset()) * 60000).getTime());
        
        // console.log(this.task);

        let data = JSON.stringify({
        query: `mutation {
          create_timesheet_item(data: {
            id: "${this.task.id}",
            date: "${this.task.date}",
            dateTime: ${this.task.dateTime},
            project: "${this.task.project}",
            department: "${this.task.department}",
            hours: "${this.task.hours}",
            notes: "${this.task.notes}",
            userId: "${this.task.userId}"
            status: "${this.task.status}"
          })
          {
            id
            date
            dateTime 
            department
            project
            userId
            hours
            notes
            user_created{
                id
                first_name
                last_name
            }
            user_updated{
                id
                first_name
                last_name
            }
            status
          }
        }`,
        variables: {}
      });

      // let token = this.$cookies.get('directus_access_token')

      // let config = {
      //   method: 'post',
      //   url: `https://5mee2e5z.directus.app/graphql`,
      //   headers: { 
      //     'Authorization': `Bearer ${token}`, 
      //     'Content-Type': 'application/json'
      //   },
      //   data : data
      // };

      // axios(config)
      // .then(function (response) {
      //   console.log(response);
      //   // that.$refs.form.reset();
      //   that.$refs.form.resetValidation();
      //   that.dateUnformated = new Date((new Date(that.date).getTime()) + (new Date(that.date).getTimezoneOffset()) * 60000);
      //   that.dialogClose();
      // })
      // .catch(function (error) {
      //   console.log(error);
      // });



      }



      
    },
    getTask(id){
      
    },
    async updateTask(id){
      if (this.$refs.form.validate()) {
        // let tasks = JSON.parse(localStorage.getItem('tasks'));
        // let index = tasks.findIndex((tsk) => (tsk.id == id));
        // this.task.id = id;
        this.task.date = String(new Date((new Date(this.date).getTime()) + (new Date(this.date).getTimezoneOffset()) * 60000).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"}));
        this.task.dateTime = parseFloat(new Date((new Date(this.date).getTime()) + (new Date(this.date).getTimezoneOffset()) * 60000).getTime());

        let that = this
        

        
        // console.log(this.task);

        let data = JSON.stringify({
        query: `mutation {
          update_timesheet_item(id:"${this.task.id}" ,data: {
            id:"${this.task.id}"
            date: "${this.task.date}",
            dateTime: ${this.task.dateTime},
            project: "${this.task.project}",
            department: "${this.task.department}",
            hours: "${this.task.hours}",
            notes: "${this.task.notes}",
            userId: "${this.task.userId}"
            status: "${this.task.status}"
          })
          {
            id
            date
            dateTime 
            department
            project
            userId
            hours
            notes
            user_created{
                id
                first_name
                last_name
            }
            user_updated{
                id
                first_name
                last_name
            }
            status
          }
        }`,
        variables: {}
      });

      // let token = this.$cookies.get('directus_access_token')

      let config = {
        method: 'post',
        url: `https://5mee2e5z.directus.app/graphql`,
        headers: { 
          'Authorization': `Bearer ${token}`, 
          'Content-Type': 'application/json'
        },
        data : data
      };

      // axios(config)
      // .then(function (response) {
      //   console.log(response);
      //   // that.$refs.form.reset();
      //   that.dateUnformated = new Date((new Date(that.date).getTime()) + (new Date(that.date).getTimezoneOffset()) * 60000);
      //   that.$refs.form.resetValidation();
      //   that.dialogClose();
      // })
      // .catch(function (error) {
      //   console.log(error);
      // });

      }
    },
    async deleteTask(){
      let id = this.deleteId;
      let that = this
        let data = JSON.stringify({
        query: `mutation {
          delete_timesheet_item(id:"${id}")
          {
            id
          }
        }`,
        variables: {}
      });

      // let token = this.$cookies.get('directus_access_token')

      let config = {
        method: 'post',
        url: `https://5mee2e5z.directus.app/graphql`,
        headers: { 
          'Authorization': `Bearer ${token}`, 
          'Content-Type': 'application/json'
        },
        data : data
      };

      // axios(config)
      // .then(function (response) {
      //   console.log(response);
      //   // that.$refs.form.reset();
      //   that.$refs.form.resetValidation();
      //   that.dialogDelete = false;
      //   that.dialogClose();
      // })
      // .catch(function (error) {
      //   console.log(error);
      // });
      
    },
    tabChange (id) {
      // console.log(id);
      let currTab = this.tabItems.filter((tab) => (tab.id == id))[0] ;
      if(currTab.date != null){
        this.dateUnformated = new Date(currTab.date).toString();
        // console.log(currTab,new Date(currTab.date).toString());
      }
    },
    projectList() {
      
      let that = this;

      // let token = this.$cookies.get('directus_access_token')
      let data = JSON.stringify({
        query: `query {
          project {
            id
            pcode
            pname
          }
        }`,
        variables: {}
      });

      // let config = {
      //   method: 'post',
      //   url: `https://5mee2e5z.directus.app/graphql`,
      //   headers: { 
      //     'Authorization': `Bearer ${token}`, 
      //     'Content-Type': 'application/json'
      //   },
      //   data : data
      // };

      // axios(config)
      // .then(function ({data}) {
      //   that.projects = [];
      //   if(data.data.project.length > 0) {
      //     try {
      //       for (const doc of data.data.project) {
              
      //         that.projects.push(doc);
      //       }
            
      //     } catch (err) {}
      //   }
      //   // console.log(that.projects);
      // })
      // .catch(function (error) {
      //   console.log(error);
      // });
    },
    departmentList() {
      let that = this;
      // let token = this.$cookies.get('directus_access_token')
      let data = JSON.stringify({
        query: `query {
          department {
            id
            dname
          }
        }`,
        variables: {}
      });

      // let config = {
      //   method: 'post',
      //   url: `https://5mee2e5z.directus.app/graphql`,
      //   headers: { 
      //     'Authorization': `Bearer ${token}`, 
      //     'Content-Type': 'application/json'
      //   },
      //   data : data
      // };

      // axios(config)
      // .then(function ({data}) {
      //   that.departments = [];
      //   if(data.data.department.length > 0) {
      //     try {
      //       for (const doc of data.data.department) {
              
      //         that.departments.push(doc);
      //       }
            
      //     } catch (err) {}
      //   }
      //   // console.log(that.departments);
      // })
      // .catch(function (error) {
      //   console.log(error);
      // });
        

    },
    deleteItem(id) {
      this.dialogDelete = true;
      this.deleteId = id;
    },
    closeDelete() {
      this.deleteId = '';
      this.dialogDelete = false
    },
    timeConvert(){ 
      let number = this.task.hours;
      
      if(number.includes(':')){
        // console.log(number);
       this.task.hours = number; 
      }
      else{
        // console.log(number);
        // Check sign of given number
      var sign = (number >= 0) ? 1 : -1;

      // Set positive value of number of sign negative
      number = number * sign;

      // Separate the int from the decimal part
      var hour = Math.floor(number);
      var decpart = number - hour;

      var min = 1 / 60;
      // Round to nearest minute
      decpart = min * Math.round(decpart / min);

      var minute = Math.floor(decpart * 60) + '';

      // Add padding if need
      // if (minute.length < 2) {
      // minute = '0' + minute; 
      // }

      // Add Sign in final result
      sign = sign == 1 ? '' : '-';

      // Concate hours and minutes
      this.task.hours = sign + this.pad(hour) + ':' + this.pad(minute);
    
        }
      
    },
  },
  watch: {
    dateUnformated: function (val, oldVal) {
      // console.log(val);
      this.setTabItems();
      this.date = new Date((new Date(val).getTime()) - (new Date(val).getTimezoneOffset()) * 60000).toISOString().substr(0, 10);
      // this.dateShow = new Date(val).toLocaleDateString('en-us', { weekday:"long", year:"numeric", month:"short", day:"numeric"});
      // console.log(this.date,this.tabItems);
      // console.log("hello",val);

    },
    // date: function (val, oldVal) {
    //   // console.log(val,this.dateUnformated);
    // }
  },
	
};
</script>
<style scoped>
.v-card {
  min-width: 100%;
  max-width: 100%;
}
.v-tabs.horizontal {
    justify-content: space-between;
    display: flex;
    flex-direction: row;
}
@media screen and (max-width:767px) {
  .v-tab.horizontal {
    padding: 15px !important;
  }
}
@media screen and (min-width:768px) {
  .v-card {
    display: flex;
    justify-content: space-between;
  }
  .v-card:first-child{
    flex: 0 0 80%;
    max-width: 80%;
  }
}

</style>
