<template>
  <div class="office">
    <div class="image-container" ref="imageContainer" @dblclick="addWorksplaces">
      <img :src="plan.img" alt="Office Plan" v-if="plan.img" class="plan" ref="Plan" />
      <img v-for="(worksplace) in worksplaces" :key="worksplace.id" :src="worksplace.img" :style="worksplace.style" class="worksplace" @click.prevent="removeWorksplaces(worksplace.id)"/>
      </div>
    <div class="info">
      <div class="menu">
        <h3>Меню</h3>
        <p>Имя плана: {{ plan.name }}</p>
      </div>
      <div class="plans">
        <h3>Планы</h3>
        <ul class="plan_buttons">
          <button class="plan_button" v-for="(plan) in plans" :key="plan.id" @click="changePlan(plan)">{{ plan.name }} </button>
        </ul>
      </div>
      <div class="users">
        <h3>Пользователи</h3>
        <ul>
          <li v-for="(user, index) in users" :key="index">{{ user.username }} {{ user.first_name }} {{ user.last_name }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'OfficeMap',
  data() {
    return {
      cursorX: 0,
      cursorY: 0,
      plan: {
        id: "0",
        name: "test",
        img: "",
        width: 0,
        height: 0,
      },
      plans:[],
      users: [],
      worksplaces: [],
      widthPlan: 0,
      heightPlan: 0,
      id_count: 0,
      relativeY: 0,
      relativeX: 0,
    };
  },
  created() {
    this.loadPlans().then((id) => {
      this.loadPlan(id).then(() => {
        this.loadUsers().then(() => {
          this.loadworksplaces(id);
        });
      });
    });
  },
  mounted() {
    window.addEventListener('resize', this.handleResize);
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.handleResize);
  },
  methods: {
    async loadPlan(id) {
      try {
        const response = await axios.get(`http://127.0.0.1:8000/plans/${id}/`, {
          headers: {
            'Content-Type': 'application/json',
          },
        });

        this.plan = response.data;
        this.$nextTick(this.handleResize);
        return response;
      } catch (error) {
        console.error('Error loading plan:', error);
      }
    },
    async loadUsers() {
      try {
        const response = await axios.get('http://127.0.0.1:8000/users/', {
          headers: {
            'Content-Type': 'application/json',
          },
        });

        this.users = response.data;
        return response;
      } catch (error) {
        console.error('Error loading users:', error);
      }
    },
    async loadPlans() {
      try {
        const response = await axios.get('http://127.0.0.1:8000/plans/', {
          headers: {
            'Content-Type': 'application/json',
          },
        });

        this.plans = response.data;
        return this.plans[0].id;
      } catch (error) {
        console.error('Error loading plans:', error);
      }
    },
    async loadworksplaces(planId) {
      try {
        const response = await axios.get('http://127.0.0.1:8000/workplaces/', {
          headers: {
            'Content-Type': 'application/json',
          },
        });

        const allWorkplaces = response.data;

        // Filter workplaces based on planId
        const filteredWorkplaces = allWorkplaces.filter(workspace => workspace.plan === planId);

        // Clear existing workplaces
        this.worksplaces = [];

        // Use $nextTick to ensure the DOM has been updated
        this.$nextTick(() => {
          // Iterate over filtered workplaces and update the worksplaces array
          filteredWorkplaces.forEach((workspace) => {
            let viewX = Math.trunc((workspace.x * this.widthPlan) / this.plan.width);
            let viewY = Math.trunc((workspace.y * this.heightPlan) / this.plan.height);

            viewX = ((viewX - 15) / this.widthPlan) * 100;
            viewY = ((viewY - 15) / this.heightPlan) * 100;

            const workspace_elem = {
              id: workspace.id,
              img: require('@/assets/computer-svgrepo-com.svg'),
              style: {
                left: `${viewX}%`,
                top: `${viewY}%`,
                zIndex: this.worksplaces.length + 1,
              },
            };

            this.worksplaces.push(workspace_elem);
          });
        });
      } catch (error) {
        console.error('Error loading worksplaces:', error);
      }
    },
    async changePlan(plan) {
      try {
        const response = await this.loadPlan(plan.id);
        this.plan = response.data;
      } catch (error) {
        console.error('Error changing plan:', error);
      } finally { 
        this.loadworksplaces(this.plan.id);
      }
    },
    handleResize() {
      this.width = window.innerWidth;
      this.height = window.innerHeight;
      this.widthPlan = this.$refs.Plan.width;
      this.heightPlan = this.$refs.Plan.height;
    },
    zoomIn() {
      this.scale += 0.1;
    },
    zoomOut() {
      if (this.scale > 0.1) {
        this.scale -= 0.1;
      }
    },
    updateCursorPosition(event) {
      this.cursorX = event.offsetX;
      this.cursorY = event.offsetY;
    },
    handleWheel(event) {
      event.preventDefault();

      const scaleFactor = event.deltaY > 0 ? 1 / 1.1 : 1.1;

      const deltaX = (this.cursorX - this.translateX) * (1 - scaleFactor);
      const deltaY = (this.cursorY - this.translateY) * (1 - scaleFactor);

      this.scale *= scaleFactor;

      this.translateX += deltaX;
      this.translateY += deltaY;
    },
    async addWorksplaces(event) {
      this.relativeX = event.offsetX;
      this.relativeY = event.offsetY;

      this.id_count++;

      const originX = Math.trunc((this.plan.width * this.relativeX) / this.widthPlan);
      const originY = Math.trunc((this.plan.height * this.relativeY) / this.heightPlan);

      try {
        const response = await axios.post(`http://127.0.0.1:8000/workplaces/`, {
          x: originX,
          y: originY,
          plan: this.plan.id,
          employee: null,
        });

        const workspace = response.data;

        let viewX = Math.trunc((workspace.x * this.widthPlan) / this.plan.width);
        let viewY = Math.trunc((workspace.y * this.heightPlan) / this.plan.height);

        viewX = ((viewX - 15) / this.widthPlan) * 100;
        viewY = ((viewY - 15) / this.heightPlan) * 100;

        const workspace_elem = {
          id: workspace.id,
          img: require('@/assets/computer-svgrepo-com.svg'),
          style: {
            left: `${viewX}%`,
            top: `${viewY}%`,
            zIndex: this.worksplaces.length + 1,
          },
        };

        this.worksplaces.push(workspace_elem);
      } catch (error) {
        console.error('Error adding worksplace:', error);
      }
    },
    async removeWorksplaces(id) {
      try {
        await axios.delete(`http://127.0.0.1:8000/workplaces/${id}/`);

        const indexToRemove = this.worksplaces.findIndex(workspace => workspace.id === id);

        if (indexToRemove !== -1) {
          this.worksplaces.splice(indexToRemove, 1);
        }
      } catch (error) {
        console.error('Error adding worksplace:', error);
      }
    },
  },
};
</script>

<style scoped>
.office {
  max-width: 100%;
  width: 100%;
  box-sizing: border-box;
}

.image-container {
  overflow: hidden;
  position: relative;
  margin: 10px;
}

.plan {
  box-sizing: border-box;
  width: 100%;
  background-repeat: no-repeat;
  background-size: contain;
  background-position: center center;
  transition: transform 0.3s ease;
}

.menu {
}

.info {
  display: flex;
    width: 100%;
    justify-content: space-evenly;
}


.worksplace {
  width: 30px;
  height: 30px;
  cursor: pointer;
  position: absolute;
}

.worksplace:hover {
  background-color: rgb(170, 170, 170, 0.7);
}

.plan_buttons{
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0;
}

.plan_button{
  width: 100%;
  margin: 3px;
}

h3 {
  padding: 0;
  margin: 3px;
}
</style>
