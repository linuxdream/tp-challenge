<template>
  <v-container>
    <v-row cols="12">
      <v-col>
        <v-card>
          <v-card-title class="text-center justify-center py-6 text-h4"
            >Wait Time Simulation Project</v-card-title
          >
          <v-card-text>
            This project runs a simulation of an ice cream shop and estimates
            the wait time for a particular Golden Customer that arrives at the
            end of the day. The time to make an ice cream cone is sampled from
            the normal distribution based on the settings below (mean and
            standard deviation) and the arrival timing of customer is drawn from
            multiple samples from the exponential distribution.
          </v-card-text>
          <v-card-text>
            The pseudo code is as such:

            <ol>
              <li>
                Since customer arrival times serve as the basis of all further
                work, customer arrival times are sampled until there are enough
                to fill the time window.
              </li>
              <li>
                Then, for every customer in the arrival list, we sample how long
                it will take the ice cream maker to make that customer's cone.
              </li>
              <li>
                We then calculate the time until the maker can start the next
                cone which is the current wait time (zero for the first
                customer) plus the time it takes for the ice cream maker to make
                the cone.
              </li>
              <li>
                Next, we update the wait time by taking the time it takes for
                the ice cream maker to start the next cone and subtracting the
                arrival time of the current customer. If that number is
                negative, then the wait time for the next customer is zero.
              </li>
              <li>
                The final wait time value at the end of the time window is the
                number of minutes, for this round, that our Golden Customer will
                need to wait until the ice cream maker can start their cone.
              </li>
              <li>
                We then repeat the above time window (day) simulation the
                requested number of times storing the daily averages across all
                simulations to give us the completed simulation averages.
              </li>
            </ol>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <v-alert v-if="notification.display" :type="notification.type">{{
      this.notification.message
    }}</v-alert>

    <v-row cols="12">
      <v-col cols="5">
        <v-card>
          <v-card-title class="text-center justify-center py-6"
            >Simulation Settings
          </v-card-title>
          <v-subheader>Time Window</v-subheader>
          <v-card-text>
            <v-slider
              v-model="simSettings.timeWindow"
              :min="simSettingsLimits.timeWindow.min"
              :max="simSettingsLimits.timeWindow.max"
              hint="How many hours from opening until the target customer arrives."
              thumb-label="always"
              persistent-hint
            >
            </v-slider>
          </v-card-text>

          <v-subheader>Average Cone-Making Time</v-subheader>
          <v-card-text>
            <v-slider
              v-model="simSettings.makeTime"
              :min="simSettingsLimits.makeTime.min"
              :max="simSettingsLimits.makeTime.max"
              hint="How many minutes, on average, it takes to make a ice cream cone."
              thumb-label="always"
              persistent-hint
            >
            </v-slider>
          </v-card-text>

          <v-subheader>Cone-Making Std</v-subheader>
          <v-card-text>
            <v-slider
              v-model="simSettings.std"
              :min="simSettingsLimits.std.min"
              :max="simSettingsLimits.std.max"
              hint="Standard deviation in minutes of cone-making time."
              thumb-label="always"
              persistent-hint
            >
            </v-slider>
          </v-card-text>

          <v-subheader>New Customer Arrivals</v-subheader>
          <v-card-text>
            <v-slider
              v-model="simSettings.newCustomerTime"
              :min="simSettingsLimits.newCustomerTime.min"
              :max="simSettingsLimits.newCustomerTime.max"
              hint="How many minutes, on average, a new customer arrives."
              thumb-label="always"
              persistent-hint
            >
            </v-slider>
          </v-card-text>

          <v-subheader>Number of Simulations</v-subheader>
          <v-card-text>
            <v-slider
              v-model="simSettings.simTimes"
              :min="simSettingsLimits.simTimes.min"
              :max="simSettingsLimits.simTimes.max"
              hint="How many times to run the simulation."
              thumb-label="always"
              persistent-hint
            >
            </v-slider>
          </v-card-text>

          <v-card-actions>
            <v-btn color="success" class="mr-4" @click="run">
              Run Simulation
            </v-btn>
            <v-btn color="warning" class="mr-4" @click="resetSettings">
              Reset
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>

      <v-col cols="7" class="mx-auto" v-if="finalSimResults.waitTimes.length">
        <!-- Numbers -->
        <v-card>
          <v-card-title class="text-center justify-center py-6"
            >Results After {{ simSettings.simTimes }} Tests</v-card-title
          >
          <div class="d-flex">
            <v-col cols="6">
              <v-card class="mx-auto" outlined>
                <v-list-item>
                  <v-list-item-content>
                    <div class="overline mb-4 text-center">
                      Avg. Golden Customer Wait Time
                    </div>
                    <v-list-item-title class="headline mb-1 text-center">
                      {{ goldenCustomerWaitTimeAvg }} minutes
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </v-card>
            </v-col>

            <v-col cols="6">
              <v-card class="mx-auto" outlined>
                <v-list-item>
                  <v-list-item-content>
                    <div class="overline mb-4 text-center">
                      Avg. Customer Wait Time
                    </div>
                    <v-list-item-title class="headline mb-1 text-center">
                      {{ customerWaitTimeAvg }} minutes
                    </v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </v-card>
            </v-col>
          </div>

          <!-- Charts -->
          <template>
            <v-card flat>
              <v-container>
                <div id="goldenWaitTimeChart"></div>
                <div id="waitTimeChart"></div>
                <div id="arrivalTimeChart"></div>
              </v-container>
            </v-card>
          </template>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { jStat } from "jstat";
import Plotly from "plotly.js";

/**
 * Define some declarations. These would typically
 * be in a separate file but placing here for ease
 * of demonstration.
 */
interface SimResults {
  waitTimes: Array<number>;
}

interface FinalSimResults extends SimResults {
  goldCustomerWaitTimes: Array<number>;
}

interface NMapping {
  [propertyName: string]: number;
}

interface MinMax {
  min: number;
  max: number;
}

interface SimSettings extends NMapping {
  timeWindow: number;
  makeTime: number;
  std: number;
  newCustomerTime: number;
  simTimes: number;
}

interface SimSettingsLimits {
  [propretyName: string]: MinMax;
}

interface Notification {
  display: boolean;
  type: NotificationType;
  message: string;
}

enum NotificationType {
  Success = "success",
  Error = "error",
}

/**
 * Main IceCream component class.
 */
@Component
export default class IceCream extends Vue {
  /**
   * Set up some parameter limits that we can use
   * later for validation.
   */
  simSettingsLimits: SimSettingsLimits = {
    timeWindow: {
      min: 1,
      max: 10,
    },
    makeTime: { min: 1, max: 10 },
    std: {
      min: 1,
      max: 10,
    },
    newCustomerTime: {
      min: 1,
      max: 10,
    },
    simTimes: {
      min: 1,
      max: 2000,
    },
  };

  tabs = null;

  notification: Notification = {
    display: false,
    type: NotificationType.Success,
    message: "",
  };

  /**
   * Main simulation params tied to the form model.
   */
  simSettings: SimSettings = this._getEmptySimSettings();

  /**
   * Object to store final results after all simulations have run.
   */
  finalSimResults: FinalSimResults = this._getEmptyFinalSimResults();

  /**
   * Let the user reset the settings to defaults.
   */
  resetSettings(): void {
    this.simSettings = this._getEmptySimSettings();
    this.finalSimResults = this._getEmptyFinalSimResults();
  }

  /**
   * Primafy function that run the simulation.
   */
  async run(): Promise<void> {
    /**
     * Do validation since we're doing math on
     * user-selected values.
     */
    if (!this._isValid()) {
      this._triggerNotification(
        NotificationType.Error,
        "There is a form validation error."
      );

      return;
    }

    /**
     * Reset the finalSimResults so we're not aggregating.
     */
    this.finalSimResults = this._getEmptyFinalSimResults();

    /**
     * Loop through the number of simulations.
     */
    for (let i = 0; i < this.simSettings.simTimes; i++) {
      /**
       * Run a simulation.
       */
      const localSimResults: SimResults = this._runSimulation(this.simSettings);

      /**
       * Create averages and save them for this sim.
       */
      this.finalSimResults.goldCustomerWaitTimes.push(
        localSimResults.waitTimes[localSimResults.waitTimes.length - 1]
      );

      this.finalSimResults.waitTimes = this.finalSimResults.waitTimes.concat(
        localSimResults.waitTimes
      );
    }

    /**
     * Wait for the v-if to render chart elements.
     */
    await this.$nextTick();

    /**
     * Draw all the charts.
     */
    this._drawCharts();

    /**
     * Async function should return a promise.
     */
    return Promise.resolve();
  }

  /**
   * Getters
   */
  get goldenCustomerWaitTimeAvg(): number {
    return this._getRoundedAverage(
      this?.finalSimResults?.goldCustomerWaitTimes
    );
  }

  get customerWaitTimeAvg(): number {
    return this._getRoundedAverage(this?.finalSimResults?.waitTimes);
  }

  _runSimulation(simSettings: SimSettings): SimResults {
    /**
     * Store results the simulation.
     */
    const simResults: SimResults = {
      waitTimes: [],
    };

    /**
     * Calculate how many minutes in the simulation.
     */
    const totalMinutes = simSettings.timeWindow * 60;

    /**
     * Frontload simulated customers for the simulation time
     * so we know where the special guest is and calculate
     * wait time.
     */
    const localCustomerTimes = this._getCustomersForTheDay(
      totalMinutes,
      simSettings.newCustomerTime
    );

    /**
     * Init the time tracker. This will be how long the
     * next customer must wait before the maker starts
     * their cone.
     */
    let waitTime = 0;

    /**
     * Go through the customers in order and sample
     * cone-making rate. Track the time overlap to see
     * how long the special customer will wait.
     */
    for (let b = 0; b < localCustomerTimes.length; b++) {
      /**
       * Pick from random normal for time to make the cone
       */
      const makerTime: number = jStat.normal.sample(
        simSettings.makeTime,
        simSettings.std
      );

      /**
       * Calculate the time until next possible start.
       */
      const timeTillNextStart = waitTime + makerTime;

      /**
       * Localize the current customer time as we might
       * change it if this is the first customer since
       * it affects next customer wait times.
       */
      let currentCustomerArrivalTime = localCustomerTimes[b];
      if (b == 0) {
        currentCustomerArrivalTime = 0;
      }

      /**
       * The wait time for the current customer is the
       * difference between the time until the next
       * cone starts and the current customer's arrival.
       * If that is negative then there is no wait.
       */
      waitTime = Math.max(timeTillNextStart - currentCustomerArrivalTime, 0);

      simResults.waitTimes.push(waitTime);
    }

    return simResults;
  }

  _drawCharts(): void {
    /**
     * Draw the Golden Wait Times
     */
    Plotly.newPlot(
      "goldenWaitTimeChart",
      [
        {
          x: this.finalSimResults.goldCustomerWaitTimes,
          type: "histogram",
          xbins: {
            size: 2,
          },
          marker: {
            color: "gold",
            line: {
              width: 1,
              color: "grey",
            },
          },
        },
      ],
      {
        title: "Gold Customer Wait Times (log)",
        yaxis: {
          type: "log",
          autorange: true,
          title: {
            text: "Frequency",
          },
        },
        xaxis: {
          title: {
            text: "Minutes",
          },
        },
      }
    );

    /**
     * Draw the Wait Times
     */
    Plotly.newPlot(
      "waitTimeChart",
      [
        {
          x: this.finalSimResults.waitTimes,
          type: "histogram",
          xbins: {
            size: 2,
          },
          marker: {
            line: {
              width: 1,
              color: "grey",
            },
          },
        },
      ],
      {
        title: "Customer Wait Times",
        yaxis: {
          title: {
            text: "Frequency",
          },
        },
        xaxis: {
          title: {
            text: "Minutes",
          },
        },
      }
    );
  }

  /**
   * Return a rounded average.
   */
  _getRoundedAverage(source: Array<number> = [0], fixed = 2): number {
    const val = source.length ? jStat.mean(source) : 0;

    return val.toFixed(fixed);
  }

  /**
   * Helper function to make sure settings are
   * within bounds.
   */
  _isValid(): boolean {
    return Boolean(
      Object.keys(this.simSettings).filter((setting) => {
        return (
          this.simSettings[setting] < this.simSettingsLimits[setting].min ||
          this.simSettings[setting] > this.simSettingsLimits[setting].max
        );
      })
    );
  }

  /**
   * Helper function to trigger the notification
   * message and type.
   */
  _triggerNotification(type: NotificationType, message: string): void {
    this.notification.message = message;
    this.notification.display = true;
    this.notification.type = type;
  }

  /**
   * Based on the number minutes in the day time window,
   * sample enough customers to fill the day.
   */
  _getCustomersForTheDay(
    maxTimeMinutes: number,
    lambda: number
  ): Array<number> {
    let customerTimesAcc = 0;
    const customerTimes = [];

    /**
     * Keep getting customers until we reach max time
     */
    while (customerTimesAcc < maxTimeMinutes) {
      /**
       * Pick from exponential distributon around lambda rate.
       */
      const customerArrivalTime: number = jStat.exponential.sample(1 / lambda);

      /**
       * Save arrival time for later
       */
      customerTimes.push(customerArrivalTime);

      /**
       * Add to our accumulator so we know when to stop.
       */
      customerTimesAcc += customerArrivalTime;
    }

    return customerTimes;
  }

  /**
   * Give me a clean finalSimResults object.
   */
  _getEmptyFinalSimResults(): FinalSimResults {
    return {
      goldCustomerWaitTimes: [],
      waitTimes: [],
    };
  }

  /**
   * Give me a clean simSettings object.
   */
  _getEmptySimSettings(): SimSettings {
    return {
      timeWindow: 7,
      makeTime: 7,
      std: 1,
      newCustomerTime: 7,
      simTimes: 1001,
    };
  }
}
</script>
