<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Activity Points & Strikes Checker</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Segoe+UI:wght@400;600;700&display=swap');
  :root {
    --color-bg: #ffffff;
    --color-text-body: #6b7280;
    --color-text-dark: #000000;
    --color-primary: #000000;
    --color-primary-hover: #1e1e1e;
    --border-radius: 0.75rem; /* 12px */
    --shadow-light: 0 4px 12px rgb(0 0 0 / 0.08);
  }
  * {
    box-sizing: border-box;
  }
  body {
    margin: 0;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background-color: var(--color-bg);
    color: var(--color-text-body);
    line-height: 1.5;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
  main {
    max-width: 1200px;
    margin: 4rem auto 6rem;
    padding: 0 1rem;
    display: flex;
    flex-direction: column;
    gap: 3rem;
  }
  header {
    text-align: center;
  }
  header h1 {
    font-weight: 700;
    font-size: 3rem; /* 48px */
    color: var(--color-text-dark);
    margin-bottom: 0.5rem;
  }
  header p {
    font-size: 1.125rem; /* 18px */
    color: var(--color-text-body);
    max-width: 600px;
    margin: 0 auto;
  }
  section.card {
    background: var(--color-bg);
    border-radius: var(--border-radius);
    box-shadow: var(--shadow-light);
    padding: 2rem 2.5rem;
  }
  form {
    display: grid;
    grid-template-columns: 1fr 2fr;
    gap: 1.5rem 2rem;
    align-items: center;
  }
  label {
    font-weight: 600;
    color: var(--color-text-body);
    font-size: 1rem;
  }
  input[type="text"] {
    font-family: inherit;
    font-size: 1rem;
    padding: 0.5rem 0.75rem;
    border: 1.5px solid #d1d5db;
    border-radius: var(--border-radius);
    color: var(--color-text-dark);
    transition: border-color 0.3s ease;
  }
  input[type="text"]:focus {
    outline: none;
    border-color: var(--color-primary);
    box-shadow: 0 0 0 3px rgba(0,0,0,0.12);
  }
  button {
    grid-column: 1 / -1;
    justify-self: center;
    font-family: 'Segoe UI Semibold', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    font-weight: 700;
    font-size: 1.125rem;
    padding: 1.25rem 3rem;
    border: none;
    border-radius: 1rem;
    background-color: var(--color-primary);
    color: white;
    cursor: pointer;
    transition: background-color 0.25s ease;
    user-select: none;
    box-shadow: 0 6px 12px rgba(0,0,0,0.16);
  }
  button:hover,
  button:focus-visible {
    background-color: var(--color-primary-hover);
  }
  table {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0 8px;
    font-size: 1rem;
    border-radius: var(--border-radius);
  }
  thead {
    color: var(--color-text-dark);
    font-weight: 700;
    font-size: 1rem;
  }
  thead th {
    text-align: left;
    padding: 0.5rem 1rem;
    background: transparent;
  }
  tbody tr {
    background: #f9fafb;
    box-shadow: var(--shadow-light);
    border-radius: var(--border-radius);
  }
  tbody tr td {
    padding: 0.75rem 1rem;
    vertical-align: middle;
    color: var(--color-text-dark);
  }
  tbody tr td.center {
    text-align: center;
  }
  .legend {
    max-width: 650px;
    margin: 0 auto;
    color: var(--color-text-body);
    font-size: 0.875rem;
    padding-top: 0.75rem;
    text-align: center;
    line-height: 1.4;
  }
  @media (max-width: 720px) {
    form {
      grid-template-columns: 1fr;
    }
    label {
      margin-bottom: 0.25rem;
    }
    button {
      width: 100%;
    }
  }

</style>
</head>
<body>
<main>
  <header>
    <h1>Activity Points &amp; Strikes Checker</h1>
    <p>Enter usernames and their activity points & strikes to check quotas and updated strike status.</p>
  </header>

  <section class="card" aria-label="User input form">
    <form id="quotaForm" novalidate>
      <label for="usernames">Usernames (comma separated):</label>
      <input type="text" id="usernames" name="usernames" placeholder="e.g. Alice,Bob,Charlie" autocomplete="off" required />
      <label for="ap">Activity Points (comma separated):</label>
      <input type="text" id="ap" name="ap" placeholder="e.g. 0/4,1/4,2/4" autocomplete="off" required />
      <label for="strikes">Strikes (0-3, comma separated):</label>
      <input type="text" id="strikes" name="strikes" placeholder="e.g. 0,1,2" autocomplete="off" required />
      <button type="submit" aria-label="Check Quota">Check Quota</button>
    </form>
  </section>

  <section class="card" aria-label="Results table">
    <table id="resultsTable" role="table" aria-live="polite" aria-relevant="all">
      <thead>
        <tr>
          <th scope="col">Username</th>
          <th scope="col" class="center">Activity Points</th>
          <th scope="col" class="center">Strikes</th>
        </tr>
      </thead>
      <tbody>
        <!-- Results injected here -->
      </tbody>
    </table>
  </section>

  <p class="legend" aria-label="Legend">
    Legend: Strikes = 1st (lowest), 2nd (medium), 3rd (highest / exiled), 0 = no strike.<br />
    Quota AP is fixed at <strong>4</strong>.
  </p>
</main>

<script>
  (() => {
    const QUOTA_AP = 4;

    const form = document.getElementById('quotaForm');
    const usernamesInput = document.getElementById('usernames');
    const apInput = document.getElementById('ap');
    const strikesInput = document.getElementById('strikes');
    const resultsTableBody = document.querySelector('#resultsTable tbody');

    function showError(message) {
      alert(message);
    }

    function strikeToLabel(strike) {
      switch (strike) {
        case 1: return '1st';
        case 2: return '2nd';
        case 3: return '3rd Exiled';
        default: return '';
      }
    }

    function clearResults() {
      resultsTableBody.innerHTML = '';
    }

    function renderResults(usernames, apValues, strikes) {
      clearResults();
      for(let i=0; i<usernames.length; i++) {
        const tr = document.createElement('tr');
        const tdUser = document.createElement('td');
        tdUser.textContent = usernames[i];
        tr.appendChild(tdUser);
        const tdAp = document.createElement('td');
        tdAp.textContent = apValues[i];
        tdAp.classList.add('center');
        tr.appendChild(tdAp);
        const tdStrike = document.createElement('td');
        tdStrike.textContent = strikeToLabel(strikes[i]);
        tdStrike.classList.add('center');
        tr.appendChild(tdStrike);
        resultsTableBody.appendChild(tr);
      }
    }

    form.addEventListener('submit', e => {
      e.preventDefault();

      const usernamesRaw = usernamesInput.value.trim();
      const apRaw = apInput.value.trim();
      const strikesRaw = strikesInput.value.trim();

      if(!usernamesRaw) {
        showError('Please enter at least one username.');
        return;
      }

      const usernames = usernamesRaw.split(/\s*,\s*/);
      if(usernames.some(u => u.length === 0)) {
        showError('Usernames cannot be empty.');
        return;
      }

      if(!apRaw) {
        showError('Please enter activity points.');
        return;
      }
      const apStrs = apRaw.split(/\s*,\s*/);
      if(apStrs.length !== usernames.length) {
        showError(`You must enter exactly ${usernames.length} activity points.`);
        return;
      }
      const apValues = [];
      for(const val of apStrs) {
        let num;
        if (val.includes('/')) {
          const parts = val.split('/');
          if (parts.length === 2) {
            num = parseInt(parts[0], 10);
          } else {
            num = NaN; // Invalid format
          }
        } else {
          num = parseInt(val, 10);
        }

        if(isNaN(num) || num < 0) {
          showError('Activity Points must be valid non-negative integers.');
          return;
        }
        apValues.push(num);
      }

      if(!strikesRaw) {
        showError('Please enter strikes.');
        return;
      }
      const strikeStrs = strikesRaw.split(/\s*,\s*/);
      if(strikeStrs.length !== usernames.length) {
        showError(`You must enter exactly ${usernames.length} strike values.`);
        return;
      }
      const strikes = [];
      for(const val of strikeStrs) {
        const num = parseInt(val, 10);
        if(isNaN(num) || num < 0 || num > 3) {
          showError('Strikes must be integers between 0 and 3.');
          return;
        }
        strikes.push(num);
      }

      // Apply strike if below quota
      for(let i=0; i<usernames.length; i++) {
        if(apValues[i] < QUOTA_AP) {
          strikes[i] = Math.min(strikes[i] + 1, 3);
        }
      }

      renderResults(usernames, apValues, strikes);
    });
  })();
</script>
</body>
</html>
