<template>
  <div class="office" @mousemove="updateCursorPosition">
    <div class="image-container" ref="imageContainer" @dblclick="addDesk">
      <img :src="plan.img" alt="Office Plan" v-if="plan.img" class="plan" ref="Plan"
        :style="{ transform: `scale(${scale}) translate(${translateX}px, ${translateY}px)` }" />
      <img v-for="(desk, index) in workspaces" :key="index" :src="desk.img" :style="desk.style" class="desk" />
    </div>
    <div class="info">
      <div class="add">
        <h3>Добавить рабочее место</h3>
        <button class="add-button" @click="addDesk">Add</button>
      </div>
      <div class="menu">
        <h3>Меню</h3>
        <p>Имя плана: {{ plan.name }}</p>
        <p>relX: {{ relativeX }}, relY: {{ relativeY }}</p>
        <!-- <p>cursorX: {{ cursorX }}, cursorY: {{ cursorY }}</p> -->
        <!-- <p>width: {{ width }}, widthPlan {{ widthPlan }}, originW {{ plan.width }}</p> -->
        <!-- <p>height: {{ height }}, heightPlan {{ heightPlan }}, originY{{ plan.height }}</p> -->
      </div>
      <div class="users">
        <h3>Пользователи</h3>
        <ul>
          <li v-for="(user, index) in users" :key="index">{{ user.username }} {{ user.first_name }} {{ user.last_name }}</li>
        </ul>
      </div>
      
    </div>
    <div class="works">
        <h3>works</h3>
        <ul>
          <li v-for="(work, index) in workspaces" :key="index">{{ work.x }} {{ work.y }} </li>
        </ul>
      </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'OfficeMap',
  data() {
    return {
      scale: 1,
      translateX: 0,
      translateY: 0,
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
      workspaces: [],
      width: window.innerWidth,
      height: window.innerHeight,
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
          this.loadWorkspaces();
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
        return response;
      } catch (error) {
        console.error('Error loading plan:', error);
      } finally {
        this.$nextTick(this.handleResize);
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
    async loadWorkspaces() {
      try {
        const response = await axios.get('http://127.0.0.1:8000/workplaces/', {
          headers: {
            'Content-Type': 'application/json',
          },
        });

        this.data = response.data;

        // Use $nextTick to ensure the DOM has been updated
        this.$nextTick(() => {
          this.data.forEach((workspace) => {
            let viewX = Math.trunc((workspace.x * this.widthPlan) / this.plan.width );
            let viewY = Math.trunc((workspace.y * this.heightPlan) / this.plan.height );

            viewX = ((viewX - 15) / this.widthPlan) * 100;
            viewY = ((viewY - 15) / this.heightPlan) * 100;

            const workspace_elem = {
              img: require('@/assets/computer-svgrepo-com.svg'),
              style: {
                left: `${viewX }%`,
                top: `${viewY }%`,
                zIndex: this.workspaces.length + 1,
              },
            };

            this.workspaces.push(workspace_elem);
          });
        });
      } catch (error) {
        console.error('Error loading workspaces:', error);
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
    async addDesk(event) {
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
          img: require('@/assets/computer-svgrepo-com.svg'),
          style: {
            left: `${viewX}%`,
            top: `${viewY}%`,
            zIndex: this.workspaces.length + 1,
          },
        };

        this.workspaces.push(workspace_elem);
      } catch (error) {
        console.error('Error adding desk:', error);
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
  width: 100%;
}

.info {
  display: flex;
}

.desk {
  width: 30px;
  height: 30px;
  cursor: pointer;
  position: absolute;
}

.desk:hover {
  background-color: rgb(170, 170, 170, 0.7);
}

h3 {
  padding: 0;
  margin: 0;
}
</style>
