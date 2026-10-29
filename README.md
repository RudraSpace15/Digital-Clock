<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Time Watch</title>
  <style>
    * { box-sizing: border-box; }

    body {
      margin: 0;
      min-height: 100vh;
      display: grid;
      place-items: center;
      padding: 20px;
      font-family: Arial, sans-serif;
      color: #f4f7ff;
      background: #101522;
    }

    .app { width: min(600px, 100%); }

    h1 { text-align: center; }

    .tabs {
      display: flex;
      gap: 8px;
      padding: 6px;
      background: #1b2334;
      border-radius: 12px;
    }

    button {
      border: 0;
      border-radius: 8px;
      padding: 12px 16px;
      color: inherit;
      background: #303b50;
      cursor: pointer;
    }

    .tab { flex: 1; }

    button.active, .primary {
      color: #101522;
      background: #78a8ff;
      font-weight: bold;
    }

    .card {
      margin-top: 16px;
      min-height: 340px;
      padding: 30px 20px;
      display: grid;
      place-items: center;
      text-align: center;
      background: #1b2334;
      border: 1px solid #303b50;
      border-radius: 20px;
    }

    .view { display: none; width: 100%; }
    .view.active { display: block; }

    .time {
      font-size: clamp(3rem, 12vw, 5rem);
      font-weight: 300;
      font-variant-numeric: tabular-nums;
    }

    .muted { color: #9aa8bf; }

    .controls {
      display: flex;
      justify-content: center;
      gap: 10px;
      margin-top: 20px;
    }

    .calendar {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 6px;
      max-width: 400px;
      margin: 16px auto;
    }

    .day {
      padding: 10px 0;
      border-radius: 8px;
    }

    .weekday { color: #9aa8bf; }
    .today { color: #101522; background: #78a8ff; font-weight: bold; }

    .month-row {
      display: flex;
      justify-content: space-between;
      align-items: center;
      max-width: 400px;
      margin: auto;
    }
  </style>
</head>
<body>
  <main class="app">
    <h1>◷ Time Watch</h1>

    <nav class="tabs">
      <button class="tab active" data-view="clock">Clock</button>
      <button class="tab" data-view="stopwatch">Stopwatch</button>
      <button class="tab" data-view="calendar">Calendar</button>
    </nav>

    <section class="card">
      <div class="view active" id="clock">
        <p class="muted">CURRENT TIME</p>
        <div class="time" id="liveTime">--:--:--</div>
        <p id="liveDate"></p>
      </div>

      <div class="view" id="stopwatch">
        <p class="muted">STOPWATCH</p>
        <div class="time" id="stopwatchTime">00:00.00</div>
        <div class="controls">
          <button class="primary" id="startStop">Start</button>
          <button id="reset">Reset</button>
        </div>
      </div>

      <div class="view" id="calendar">
        <p class="muted">CALENDAR</p>
        <div class="month-row">
          <button id="prevMonth" aria-label="Previous month">‹</button>
          <strong id="monthTitle"></strong>
          <button id="nextMonth" aria-label="Next month">›</button>
        </div>
        <div class="calendar" id="calendarGrid"></div>
      </div>
    </section>
  </main>

  <script>
    const tabs = document.querySelectorAll(".tab");

    tabs.forEach(tab => {
      tab.addEventListener("click", () => {
        tabs.forEach(item => item.classList.toggle("active", item === tab));
        document.querySelectorAll(".view").forEach(view => {
          view.classList.toggle("active", view.id === tab.dataset.view);
        });
      });
    });

    function updateClock() {
      const now = new Date();
      document.getElementById("liveTime").textContent =
        new Intl.DateTimeFormat(undefined, {
          hour: "2-digit", minute: "2-digit", second: "2-digit"
        }).format(now);

      document.getElementById("liveDate").textContent =
        new Intl.DateTimeFormat(undefined, {
          weekday: "long", year: "numeric", month: "long", day: "numeric"
        }).format(now);
    }

    updateClock();
    setInterval(updateClock, 1000);

    let elapsed = 0;
    let startTime = 0;
    let running = false;
    let timer;

    function showStopwatch() {
      const total = elapsed + (running ? performance.now() - startTime : 0);
      const minutes = Math.floor(total / 60000);
      const seconds = Math.floor(total / 1000) % 60;
      const hundredths = Math.floor(total / 10) % 100;

      document.getElementById("stopwatchTime").textContent =
        `${String(minutes).padStart(2, "0")}:${String(seconds).padStart(2, "0")}.${String(hundredths).padStart(2, "0")}`;
    }

    document.getElementById("startStop").addEventListener("click", event => {
      if (running) {
        elapsed += performance.now() - startTime;
        running = false;
        clearInterval(timer);
        event.target.textContent = "Start";
        event.target.classList.add("primary");
      } else {
        startTime = performance.now();
        running = true;
        timer = setInterval(showStopwatch, 10);
        event.target.textContent = "Stop";
        event.target.classList.remove("primary");
      }
      showStopwatch();
    });

    document.getElementById("reset").addEventListener("click", () => {
      elapsed = 0;
      if (running) startTime = performance.now();
      showStopwatch();
    });

    let shownMonth = new Date();
    const weekdays = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];

    function drawCalendar() {
      const year = shownMonth.getFullYear();
      const month = shownMonth.getMonth();
      const today = new Date();

      document.getElementById("monthTitle").textContent =
        new Intl.DateTimeFormat(undefined, { month: "long", year: "numeric" })
          .format(shownMonth);

      const grid = document.getElementById("calendarGrid");
      grid.replaceChildren();

      weekdays.forEach(name => {
        const cell = document.createElement("div");
        cell.className = "day weekday";
        cell.textContent = name;
        grid.appendChild(cell);
      });

      const firstDay = new Date(year, month, 1).getDay();
      const daysInMonth = new Date(year, month + 1, 0).getDate();

      for (let i = 0; i < firstDay; i++) {
        const blank = document.createElement("div");
        grid.appendChild(blank);
      }

      for (let day = 1; day <= daysInMonth; day++) {
        const cell = document.createElement("div");
        cell.className = "day";
        cell.textContent = day;

        if (
          year === today.getFullYear() &&
          month === today.getMonth() &&
          day === today.getDate()
        ) {
          cell.classList.add("today");
        }

        grid.appendChild(cell);
      }
    }

    document.getElementById("prevMonth").addEventListener("click", () => {
      shownMonth = new Date(shownMonth.getFullYear(), shownMonth.getMonth() - 1, 1);
      drawCalendar();
    });

    document.getElementById("nextMonth").addEventListener("click", () => {
      shownMonth = new Date(shownMonth.getFullYear(), shownMonth.getMonth() + 1, 1);
      drawCalendar();
    });

    drawCalendar();
  </script>
</body>
</html>
