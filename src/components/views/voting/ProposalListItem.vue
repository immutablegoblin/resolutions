import { ethers } from "ethers";
<template>
  <section @click="openProposal" class="proposal has-radius-md mb-5" aria-labelledby="proposal">
    <span class="proposal-type" :class="this.typeStylingData.class">
      {{ this.typeStylingData.text }}
    </span>
    <div @click="routeToProposal" class="is-flex is-flex-direction-column is-justify-content-space-between mb-2 mt-2 pt-4 pb-0 px-4">
      <div>
        <h2 class="is-size-5 has-text-mediumBlue">
          {{ this.startDate }}
        </h2>
        <h2 id="proposal-title" class="is-size-5 has-text-white mb-4">
          {{ proposal.title }}
        </h2>
        <p class="description"> {{ proposal.description }}</p>
      </div>
      <dl class="mt-5 mb-0 pb-0">
        <dt class="mt-2 mb-1 help">Creator:</dt>
        <dd class="mb-3">
          <Address :value="proposal.creator" />
        </dd>
      </dl>
    </div>
  </section>
</template>

<script>
import Address from "../address/Address.vue";
import Button from "../common/Button.vue";
import {
  getProposalTypeStyling,
  padWithZeroes,
  dateStringForTimestamp,
  getVotes,
  getResult,
  hasEnded,
} from "@/data/helpers";
import { PASSED } from "@/models/common";

export default {
  name: "ProposalListItem",
  components: {
    Address,
  },
  props: {
    assetId: {
      type: String,
      required: true,
    },
    proposal: {
      type: Object,
      required: true,
    },
    embedded: {
      type: Boolean,
      required: false,
    },
  },
  data() {
    return {
      timeRemainingString: "",
      PASSED,
    };
  },
  computed: {

    votes() {
      return getVotes(this.proposal);
    },

    startDate() {
      const startDate = new Date(this.proposal.startTimestamp * 1000);
      return `${padWithZeroes(startDate.getDate())}/${padWithZeroes(startDate.getMonth() + 1)}`;
    },

    startDateString() {
      return dateStringForTimestamp(this.proposal.startTimestamp);
    },

    endDateString() {
      return dateStringForTimestamp(this.proposal.endTimestamp);
    },

    ended() {
      return hasEnded(this.proposal);
    },

    passed() {
      return getResult(this.proposal);
    },

    typeStylingData() {
      return getProposalTypeStyling(this.proposal.type);
    },
  },
  methods: {
    setTimeRemainingCountdown() {
      clearInterval(this.countdownRef);

      this.countdownRef = setInterval(
        function () {
          let now = new Date().getTime() / 1000;

          let t = this.proposal.endTimestamp - now;

          if (t >= 0) {
            let days = Math.floor(t / (60 * 60 * 24));
            let hours = Math.floor((t % (60 * 60 * 24)) / (60 * 60));
            let mins = Math.floor((t % (60 * 60)) / 60);
            let secs = Math.floor(t % 60);

            this.timeRemainingString = `${days}d, ${hours}h, ${mins}m, ${secs}s`;
          } else {
            this.timeRemainingString = "The voting is over";
          }
        }.bind(this),
        1000
      );
    },

    openProposal() {
      this.$router.push(`/proposal/${this.proposal.id}`);
    },
  },

  created() {
    this.setTimeRemainingCountdown();
  },

  routeToProposal() {
    this.$router.push(`/proposal/${this.proposal.id}`);
  },
};
</script>

<style scoped lang="scss">
@import "../../../styles/frabric-custom.scss";
@import "../../../styles/_variables.sass";

.proposal {
  background-color: $darkGray;
  position: relative;
  min-width: 320px;
  cursor: pointer;
  transition: all 150ms;
  &:hover {
    filter: contrast(95%);
  }
}

.proposal-type {
  font-weight: 400;
  padding: 5px 10px;
  border: 2px solid white;
  position: absolute;
  top: 0;
  right: 0;
  border-radius: 0 0.5rem 0 0 !important;
}
.paper {
  border-color: #00EDC4;
  color: #00EDC4;
}

.participant {
  border-color: whitesmoke;
  color: whitesmoke;
}

.upgrade {
  border-color: #D841DE;
  color: #D841DE;
}

.outcome-box {
  color: white;
  font-weight: 600;
  border-radius: 0.5rem;
  padding: 15px 20px;
}

.description {
  display: inline-block;
  line-height: 1.2rem;
  max-height: 7.2rem;
  white-space: nowrap;
  max-width: 56ch;
  overflow: hidden;
  text-overflow: ellipsis;
}

.bottom-right-corner {
  position: absolute;
  bottom: 0;
  right: 0;
  border-radius: 0 0 0.5rem 0;
}

.passed {
  background: $mint;
}

.failed {
  background: $red;
}

.tied {
  background: $cyan;
}

</style>