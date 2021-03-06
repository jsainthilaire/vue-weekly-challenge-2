<template>
  <div>
    <Camera />
    <div>
      <button @click="addLiquidToContainer(containerA.name)">+ {{ containerA.name }}</button>
      <button @click="addLiquidToContainer(containerB.name)">+ {{ containerB.name }}</button>
    </div>
    <div class="containers">
      <SingleContainer
          v-if="shouldShow(containerA.name)"
          :waterLevel="containerA.level"
          container-name="containerA.name"
          :distance-to-bottom="containerA.distance"
          v-on:addLiquid="addLiquidToContainer">
      </SingleContainer>
      <Channel :level="channelLvl"></Channel>
      <keep-alive>
      <SingleContainer
          v-if="shouldShow(containerB.name)"
          :waterLevel="containerB.level"
          container-name="containerB.name"
          :distance-to-bottom="containerB.distance"
          v-on:addLiquid="addLiquidToContainer">
      </SingleContainer>
        </keep-alive>
    </div>
  </div>
</template>

<script>
  import SingleContainer from './SingleContainer.vue'
  import Channel from './Channel.vue'
  import Camera from './Camera.vue'

  import { EventBus, TOGGLE_CAM } from './EventBus';

  const stepLenght = 20
  const maximunVolume = 100

  export default {
    name: 'Containers',
    mounted: function () {
      EventBus.$on(TOGGLE_CAM, containerToShow => {
        this.showContainer = containerToShow
      })
    },
    beforeDestroy: function () {
      EventBus.$off(TOGGLE_CAM)
    },
    data: function () {
      return {
        containerA: {
          level: 0,
          distance: 1,
          name: 'A',
        },
        containerB: {
          level: 20,
          distance: 2,
          name: 'B',
        },
        channelLvl: 0,
        showContainer: '',
      }
    },
    methods: {
      addLiquidToContainer: function (containerName = '') {
        switch (containerName) {
          case 'A':
            this.distribute(this.containerA, this.containerB);
            break;
          case 'B':
            this.distribute(this.containerB, this.containerA);
            break;
          default:
            throw 'Container does not exist'
        }
      },
      distribute: function (movingContainer, staticContainer) {
        if (movingContainer >= maximunVolume) {
          return;
        }

        movingContainer.level += stepLenght;

        if (this.isAboveChannel(movingContainer)) {
          const waterAbove = this.getWaterAboveChannel(movingContainer);
          const projection = { level: staticContainer.level + waterAbove, distance: staticContainer.distance }
          const waterAboveProjection = this.getWaterAboveChannel(projection);

          if (waterAboveProjection <= 0) {
            // this is just to make a "nice" simulation
            const volumeToDistrute = waterAbove / 2;
            movingContainer.level -= volumeToDistrute
            this.channelLvl += volumeToDistrute

            window.setTimeout(function () {
              movingContainer.level -= volumeToDistrute
              this.channelLvl -= volumeToDistrute
              staticContainer.level += waterAbove
            }.bind(this), 100);
          } else {
            const avgToDistribute = waterAboveProjection / 3

            if (!this.isChannelFull) {
              movingContainer.level -= (avgToDistribute * 2)
              this.channelLvl += avgToDistribute
              staticContainer.level += avgToDistribute
            }
          }
        }
      },
      isAboveChannel: function ({ level, distance }) {
        const units = this.levelToUnit(level);

        return units > distance
      },
      levelToUnit: function (level) {
        return level / 20;
      },
      getWaterAboveChannel: function({ level, distance }) {
        const unitsDiference =  this.levelToUnit(level) - distance;

        return unitsDiference * 20 - this.channelLvl;
      },
      shouldShow(containerName = '') {
        return this.showContainer === '' || containerName === this.showContainer;
      },
    },
    computed: {
      isChannelFull: function () {
        return this.channelLvl >= 20
      }
    },
    components: {
      SingleContainer,
      Channel,
      Camera,
    }
  }
</script>

<style>
  .liquid {
    display: flex;
    flex-direction: column-reverse;
    width: 40px;
    height: 100px;
    background-color: blue;
    z-index: 3;
  }

  .containers {
    display: flex;
    align-items: flex-end;
  }
</style>
