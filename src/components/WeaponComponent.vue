<template>
  <div :class="['weapon-wrapper', { favorite: isFavorite }]">
    <div :class="['weapon', { label, 'coming-soon': weapon.comingSoon }]">
      <div
        :class="[
          'name',
          layout,
          {
            'gold': camouflageCompleted('Gold'),
            'diamond': camouflageCompleted('Diamond'),
            'dark-spine': camouflageCompleted('Dark Spine'),
            'dark-matter': camouflageCompleted('Dark Matter'),
            'mystic-gold': camouflageCompleted('Mystic Gold'),
            'opal': camouflageCompleted('Opal'),
            'afterlife': camouflageCompleted('Afterlife'),
            'nebula': camouflageCompleted('Nebula'),
            'gold-tiger': camouflageCompleted('Gold Tiger'),
            'kings-ransom': camouflageCompleted('King\'s Ransom'),
            'catalyst': camouflageCompleted('Catalyst'),
            'abyss': camouflageCompleted('Abyss'),
          },
        ]"
        :data-label="label"
        @dblclick="toggleWeaponCompleted(weapon, completed, progressKey)"
        v-tippy="{
          content: $t('pages.multiplayer.double_click_tooltip', {
            state: completed
              ? $t('general.reset').toLowerCase()
              : $t('general.complete').toLowerCase(),
          }),
        }">
        {{ weapon.name }}
      </div>

      <div
        v-if="!weapon.comingSoon"
        class="progress"
        :style="{
          'grid-template-rows': `${
            layout === 'list' ? 'repeat(' + Object.keys(camouflages).length - 6 + ', 1fr)' : ''
          }`,
          'grid-template-columns': `${layout === 'grid' ? 'repeat(' + 9 + ', 1fr)' : ''}`,
        }">
        <div
          v-for="camouflage in camouflages.slice(0, 9)"
          :key="camouflage.name"
          :class="['camouflage', `weapon-layout-${layout}`]"
          @click="
            toggleCamouflageCompleted(
              weapon.name,
              camouflage.name,
              camouflage.completed,
              progressKey
            )
          "
          :content="requirementTooltip(weapon, camouflage.name)"
          v-tippy="{ placement: 'bottom' }">
          <div :class="['inner', { completed: camouflage.completed }]">
            <img
              :src="`https://emilcarlsson.se/nebula/camouflages/${convertToKebabCase(
                camouflage.name
              )}.png`"
              :alt="camouflage.name"
              onerror="javascript:this.src='/military-gradient.svg'" />
            <IconComponent class="complete" name="check" fill="#10ac84" />
            <IconComponent class="remove" name="times" fill="#ee5253" />
            <div v-if="layout === 'list'" class="info">
              <span class="requirement">{{ requirementListText(weapon, camouflage.name) }}</span>
            </div>
          </div>
        </div>
      </div>

      <div
        v-if="!weapon.comingSoon"
        class="progress"
        :style="{
          'grid-template-rows': `${
            layout === 'list' ? 'repeat(' + Object.keys(camouflages).length - 9 + ', 1fr)' : ''
          }`,
          'grid-template-columns': `${layout === 'grid' ? 'repeat(' + 6 + ', 1fr)' : ''}`,
        }">
        <div
          v-for="camouflage in camouflages.slice(9, 15)"
          :key="camouflage.name"
          :class="['camouflage', `weapon-layout-${layout}`]"
          @click="
            toggleCamouflageCompleted(
              weapon.name,
              camouflage.name,
              camouflage.completed,
              progressKey
            )
          "
          :content="requirementTooltip(weapon, camouflage.name)"
          v-tippy="{ placement: 'bottom' }">
          <div :class="['inner', { completed: camouflage.completed }]">
            <img
              :src="`https://emilcarlsson.se/nebula/camouflages/${convertToKebabCase(
                camouflage.name
              )}.png`"
              :alt="camouflage.name"
              onerror="javascript:this.src='/military-gradient.svg'" />
            <IconComponent class="complete" name="check" fill="#10ac84" />
            <IconComponent class="remove" name="times" fill="#ee5253" />
            <div v-if="layout === 'list'" class="info">
              <span class="requirement">{{ requirementListText(weapon, camouflage.name) }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <IconComponent
      class="favorite-icon"
      name="star"
      :fill="isFavorite ? '#feca57' : 'gray'"
      icon-style="solid"
      size="25"
      @click="
        toggleFavorite({
          type:
            (progressKey === 'progress') | (progressKey === 'multiplayer')
              ? 'weapons'
              : progressKey,
          name: weapon.name,
        })
      "
      v-tippy="{
        content: $t('filters.toggle_favorite', {
          state: isFavorite ? $t('general.remove_from') : $t('general.add_to'),
        }),
      }" />
  </div>
</template>

<script>
import { useStore } from '@/stores/store'
import { convertToKebabCase } from '@/utils/utils'
import { mapActions, mapState } from 'pinia'

export default {
  props: {
    weapon: {
      type: Object,
      required: true,
    },

    camouflages: {
      type: Array,
      required: true,
    },

    progressKey: {
      type: String,
      required: false,
      default: 'progress',
    },
  },

  data() {
    return {
      store: useStore(),
    }
  },

  computed: {
    ...mapState(useStore, ['weaponRequirements', 'preferences']),

    layout() {
      return this.preferences.layout
    },

    progress() {
      return this.weapon.progress[this.progressKey]
    },

    completed() {
      return Object.values(this.progress).every(Boolean)
    },

    isFavorite() {
      if (!this.store) return false
      let type =
        (this.progressKey === 'progress') | (this.progressKey === 'multiplayer')
          ? 'weapons'
          : this.progressKey
      return this.store.isFavorite(type, this.weapon.name)
    },

    label() {
      if (this.weapon.comingSoon) {
        return this.$t('general.coming_soon')
      } else {
        return null
      }
    },
  },

  methods: {
    convertToKebabCase,
    ...mapActions(useStore, [
      'toggleCamouflageCompleted',
      'toggleWeaponCompleted',
      'toggleFavorite',
    ]),

    camouflageCompleted(camouflage) {
      return this.progress[camouflage] || false
    },

    requirementTooltip(weapon, camouflage) {
      let requirement = 'TBA'

      requirement = this.weaponRequirements[weapon.name][this.progressKey][camouflage]

      if (requirement && Object.keys(requirement).length > 0) {
        if (requirement.challenge) {
          return `${camouflage} - Level ${requirement.level} - ${this.translateChallenge(
            requirement.challenge
          )}`
        } else {
          return `${camouflage} - ${this.translateChallenge(requirement)}`
        }
      } else {
        return `${camouflage} - ${requirement || 'TBA'}`
      }
    },

    requirementListText(weapon, camouflage) {
      let requirement = 'TBA'

      requirement = this.weaponRequirements[weapon.name][this.progressKey][camouflage]

      if (requirement && Object.keys(requirement).length > 0) {
        if (requirement.challenge) {
          return `${this.translateChallenge(requirement.challenge)}`
        } else {
          return `${this.translateChallenge(requirement)}`
        }
      } else {
        return requirement || 'TBA'
      }
    },

    translateChallenge(challenge) {
      if (!challenge) return 'TBA'

      const { amount, type, seconds, times } = challenge

      if (seconds) {
        return this.$t(`challenges.types.${type}`, { amount, seconds, times })
      } else if (times) {
        return this.$t(`challenges.types.${type}`, { amount, times })
      } else {
        return this.$t(`challenges.types.${type}`, { amount })
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.weapon-wrapper {
  position: relative;

  &.favorite .favorite-icon {
    opacity: 1 !important;
  }

  &:hover {
    .favorite-icon {
      opacity: 1;
    }
  }

  .favorite-icon {
    cursor: pointer;
    opacity: 0;
    position: absolute;
    right: 0;
    top: 0;
    transition: $transition;
    transform: translate(50%, -50%);
    z-index: 2;

    @media (max-width: $tablet) {
      opacity: 1 !important;
      transform: translate(50%, -50%) scale(1.25);
    }
  }

  .weapon {
    position: relative;

    &.label {
      &.coming-soon .name {
        background: $elevation-1-color;
        color: rgba($text-color, 0.25);
        pointer-events: none;

        &::after {
          background: $blue;
        }
      }

      .name {
        &::after {
          background: $purple;
          border: 3px solid $background-color;
          border-radius: 100px;
          bottom: 0;
          color: $text-color;
          content: attr(data-label);
          font-size: 10px;
          font-weight: 500;
          left: 50%;
          padding: 4px 6px;
          position: absolute;
          transform: translate(-50%, 50%);
          z-index: 2;
        }
      }
    }

    .name {
      $padding: 25px;
      $list-padding: 25px 10px;
      align-items: center;
      background: $elevation-3-color;
      border-radius: $border-radius;
      cursor: pointer;
      display: flex;
      font-weight: 600;
      justify-content: center;
      position: relative;
      transition: $transition;
      user-select: none;

      @media (max-width: $tablet) {
        font-size: 18px;
        padding: $list-padding;
      }

      &.gold {
        @include gold-camouflage-background;
        color: black;
      }

      &.diamond {
        @include diamond-camouflage-background;
        color: black;
      }

      &.dark-spine {
        @include dark-spine-camouflage-background;
        color: black;
      }

      &.dark-matter {
        @include dark-matter-camouflage-background;
        color: white;
      }

      &.mystic-gold {
        @include mystic-gold-camouflage-background;
        color: black;
      }

      &.opal {
        @include opal-camouflage-background;
        color: black;
      }

      &.afterlife {
        @include afterlife-camouflage-background;
        color: white;
      }

      &.nebula {
        @include nebula-camouflage-background;
        color: white;
      }

      &.gold-tiger {
        @include gold-tiger-camouflage-background;
        color: black;
      }

      &.kings-ransom {
        @include kings-ransom-camouflage-background;
        color: white;
      }

      &.catalyst {
        @include catalyst-camouflage-background;
        color: white;
      }

      &.abyss {
        @include abyss-camouflage-background;
        color: black;
      }

      &.grid {
        padding: $padding 0;
      }

      &.list {
        padding: $list-padding;
        &::after {
          bottom: -4px;
        }
      }
    }

    .progress {
      display: grid;
      gap: 5px;
      margin-top: 5px;

      .camouflage {
        position: relative;
        user-select: none;

        &.weapon-layout-grid > .inner {
          flex-direction: column;
          justify-content: center;

          &.completed > .info {
            opacity: 0.5;
          }

          img {
            height: 100%;
            object-fit: cover;
            position: relative;
            width: 100%;
            z-index: 1;
          }

          .icon-component {
            left: 50%;
            opacity: 0;
            position: absolute;
            transform: translate(-50%, -50%);
            transition: $transition;
            top: 50%;
            z-index: 2;
          }

          .info {
            padding: 8px;

            .name {
              font-size: 14px;
            }
          }
        }

        &.weapon-layout-list > .inner {
          $image-size: 30px;
          background: $elevation-1-color;
          flex-direction: row;
          height: $image-size;
          justify-content: flex-start;

          &.completed > .info {
            opacity: 0.5;
          }

          img {
            height: 100%;
            object-fit: cover;
            position: relative;
            width: $image-size;
            z-index: 1;
          }

          .icon-component {
            left: calc($image-size / 2);
            opacity: 0;
            position: absolute;
            transform: translate(-50%, -50%);
            transition: $transition;
            top: 50%;
            z-index: 2;
          }

          .info {
            padding: 0 8px;
            text-align: left;

            .requirement {
              display: block;
              font-size: 11px;
              line-height: 1.2;

              @media (max-width: $tablet) {
                font-size: 15px;
              }
            }
          }
        }

        .favorite-camouflage-icon {
          position: absolute;
          right: 0;
          top: 0;
          transform: translate(40%, -40%);
          z-index: 2;
        }

        .inner {
          align-items: center;
          background: $elevation-2-color;
          border-radius: $border-radius;
          cursor: pointer;
          display: flex;
          height: 100%;
          justify-content: center;
          overflow: hidden;
          position: relative;
          transition: $transition;
          width: 100%;

          &:hover {
            @media (min-width: $tablet) {
              img,
              p {
                opacity: 0.25;
              }

              .icon-component.complete {
                opacity: 1;
              }
            }
          }

          &.completed {
            &:hover {
              @media (min-width: $tablet) {
                .icon-component.complete {
                  opacity: 0;
                }
                .icon-component.remove {
                  opacity: 1;
                }
              }
            }

            img,
            p {
              opacity: 0.25;
            }

            .icon-component {
              &.complete {
                opacity: 1;
              }

              &.remove {
                opacity: 0;
              }
            }
          }

          &.disabled {
            cursor: not-allowed;

            &:hover {
              img,
              p {
                opacity: 1;
              }

              .icon-component {
                opacity: 0;
              }
            }
          }

          .icon-component {
            left: 50%;
            opacity: 0;
            position: absolute;
            transform: translate(-50%, -50%);
            transition: $transition;
            top: 50%;
            z-index: 2;
          }
        }
      }
    }
  }
}
</style>
