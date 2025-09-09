<!-- markdownlint-disable no-inline-html -->
<details>
  <summary style="cursor: pointer; border-radius: 50%; float: right;">&#9432;</summary>
  <div style="background: purple; margin: 0.5em 0; padding: 0.25em 0.75em;">
    <p>
      What % of profile-indicated insulin did the loop need to deliver?
    </p>
    <p>
      This calculator solves this TDD equation for <code>percent</code>:<br />
      <code>insulin = (carbs / ic + basal * hrs) * (percent / 100)</code>
    </p>
    <p>
      This is a simple check of totals, but it can be useful for quickly
      assessing how closely your basal and IC match reality, especially after a
      sudden change due to exercise or illness.
    </p>
    <p>
      <i>Note: since this is a simple calculation from TDD, it inherently assumes
      "equilibrium" at the start and end of that time. Don't use it on days
      that start or end with highs/lows/IoB/CoB.</i>
    </p>
  </div>
</details>

## Profile IC

| Start                                                        | Hours                         | g/U                                                |
|--------------------------------------------------------------|-------------------------------|----------------------------------------------------|
| <input id="ic-start-0" type="time" readonly value="00:00" /> | <span id="ic-hours-0"></span> | <input id="ic-ratio-0" type="number" step="0.1" /> |
| <input id="ic-start-1" type="time" step="3600" />            | <span id="ic-hours-1"></span> | <input id="ic-ratio-1" type="number" step="0.1" /> |
| <input id="ic-start-2" type="time" step="3600" />            | <span id="ic-hours-2"></span> | <input id="ic-ratio-2" type="number" step="0.1" /> |

**Average IC:** <span id="ic-daily"></span> g/U

## Profile Basal

**Basal rate:** Σ <input id="basal-daily" type="number" />U (daily)

## Statistics TDD

| Hours                                               | Σ                                           | Bo/Ba/% | Carbs                                     |
|-----------------------------------------------------|---------------------------------------------|---------|-------------------------------------------|
| <input id="hours" type="number" placeholder="24" /> | <input id="total-insulin" type="number" />U | -       | <input id="total-carbs" type="number" />g |

**Profile % delivered:** <span id="percent-delivered"></span>

<script>
  function squashAllTimeInputSeconds() {
    document.querySelectorAll('input[type="time"]').forEach(el =>
      el.addEventListener('input', e =>
        e.target.value = e.target.value.replace(/^(\d\d):\d\d$/, '$1:00')
      )
    );
  }

  function bindAllInputsToLocalStorage() {
    document.querySelectorAll('input:not([readonly])').forEach(el => {
      el.value = localStorage.getItem(el.id);
      el.addEventListener('input', ev => {
        localStorage.setItem(el.id, ev.target.value);
      });
    });
  }

  function calcIcScheduleRowHours(slotId) {
    const start = document.getElementById(`ic-start-${slotId}`);
    const end = document.getElementById(`ic-start-${slotId + 1}`);
    const hours = document.getElementById(`ic-hours-${slotId}`);

    const ticksPerHour = 1000 * 60 * 60;
    const startHours = isNaN(start?.valueAsNumber) ? 24 : start.valueAsNumber / ticksPerHour;
    const endHours = isNaN(end?.valueAsNumber) ? 24 : end.valueAsNumber / ticksPerHour;

    return hours.textContent = endHours >= startHours ? endHours - startHours : NaN;
  }

  function calcIcDaily() {
    const hours0 = calcIcScheduleRowHours(0);
    const hours1 = calcIcScheduleRowHours(1);
    const hours2 = calcIcScheduleRowHours(2);
    const avgIc = (
      (hours0 ? hours0 * parseFloat(document.getElementById('ic-ratio-0').value) : 0)
      + (hours1 ? hours1 * parseFloat(document.getElementById('ic-ratio-1').value) : 0)
      + (hours2 ? hours2 * parseFloat(document.getElementById('ic-ratio-2').value) : 0)
    ) / (hours0 + hours1 + hours2);
    return document.getElementById('ic-daily').textContent = Number(avgIc).toFixed(1);
  }

  function calc() {
    const ic = calcIcDaily();
    const basalHourly = parseFloat(document.getElementById('basal-daily').value) / 24;
    const hoursStr = document.getElementById('hours').value;
    const hours = hoursStr ? parseFloat(hoursStr) : 24;
    const totalInsulin = parseFloat(document.getElementById('total-insulin').value);
    const totalCarbs = parseFloat(document.getElementById('total-carbs').value);
    const percentage = 100 * totalInsulin / (totalCarbs / ic + basalHourly * hours);

    document.getElementById('percent-delivered').textContent = Number(percentage).toFixed(0);
  }

  function bindCalc() {
    document.querySelectorAll('input').forEach(el => {
      el.addEventListener('input', ev => calc());
    });
  }

  squashAllTimeInputSeconds();
  bindAllInputsToLocalStorage();
  bindCalc();
  calc();

</script>
<style>
  input:not([type=time]) {
    width: 3.5em;
  }

  span {
    color: violet;
  }

  p {
    margin-top: 0.2em;
    margin-bottom: 0.4em;
  }

  th {
    padding: 0;
  }

  td {
    padding: 0.25em 0.5em;
    white-space: nowrap;
  }

  input {
    background: #2c3815;
    color: white;
    padding: 0.25em 0.5em;
    border: 0.05em solid;
    border-color: grey;
  }

  h1,
  header h1 {
    font-size: 1.5em;
  }

  h2,
  #main_content h2 {
    margin: 0;
    font-size: 1.25em;
  }

  table {
    margin-bottom: 0.4em;
  }

  header {
    margin: 0;
    padding: 0;
    padding-bottom: 0;
  }

  header h2 {
    display: none;
  }

  #downloads {
    display: none;
  }

  #downloads .btn {
    padding: 0.3em 0.7em;
  }
</style>
