<style>

</style>

<div>
  <div class="sort-text">
    {#if (err)}
    <div class="row">
      <div class="col">
        <div class="alert alert-danger">{err}</div>
      </div>
    </div>
    {/if}
    <div class="row">
      <div class="col">
      </div>
      <div class="col form-group">
        <div class="btn-group">
          <div class="btn btn-primary" on:click|preventDefault|capture={format}>Format</div>
          <div class="btn btn-primary" on:click|preventDefault|capture={minify}>Minify</div>
          <div class="btn btn-primary" on:click|preventDefault|capture={clear}>Clear</div>
          <div class="btn btn-primary"
            disabled="{history.length === 0}"
            class:disabled={history.length === 0}
            on:click|preventDefault|capture={undo}
          >Undo</div>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col">
        <div class="form-group">
          <AceEditor
            bind:value={value}
            lang={aceConfig.lang}
            theme="clouds_midnight"
            width="100%"
            height="512"
            on:init={editorInit}
            on:input={onEditorChange}
          />
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col">
        <div class="form-group">
          <div class="dropdown dropdown-dark">
            <select class="form-control dropdown-select"
              bind:value={aceConfig.lang}
            >
              {#each aceOptions.lang as option}
                <option value={option}>
                  {option + ' (syntax highlighter)'}
                </option>
              {/each}
            </select>
          </div>
        </div>
      </div>
      <div class="col">
        <div class="form-group">
          <select class="form-control dropdown-select"
            bind:value={formatterConfig.lang}
          >
            {#each formatterOptions.lang as option}
              <option value={option}>
                {option + ' (syntax formatter)'}
              </option>
            {/each}
          </select>
        </div>
      </div>
      <div class="col">
        <div class="form-group">
          <select class="form-control dropdown-select"
            bind:value={formatterConfig.linesBetweenQueries}
          >
            {#each formatterOptions.linesBetweenQueries as option}
              <option value={option.value}>
                {option.text}
              </option>
            {/each}
          </select>
        </div>
      </div>
      <div class="col">
        <div class="form-group">
          <select class="form-control dropdown-select"
            bind:value={formatterConfig.indent}
          >
            {#each formatterOptions.indent as option}
              <option value={option.value}>
                {option.text}
              </option>
            {/each}
          </select>
        </div>
      </div>
    </div>
  </div>
</div>

<script>
  // import sqlFormatter from 'sql-formatter'
  import * as prettierSql from 'prettier-sql'
  import pick from 'lodash/pick'
  import AceEditor from 'svelte-ace-editor';
  console.log({ prettierSql })

  if (process.browser) {
    require('brace');
    require('brace/ext/language_tools')
    require('brace/mode/pgsql')
    require('brace/mode/mysql')
    require('brace/mode/sql')
    require('brace/mode/sqlserver')
    require('brace/theme/clouds_midnight')
  }

  const whitespaceRegEx = /\s+/gim
  const leftparenthesesRegEx = /\(\s+/gim
  const rightparenthesesRegEx = /\s+\)/gim

  let err = '';
  let value = '';
  let history = [];
  let maxHistory = 10;
  let formatterOptions = Object.freeze({
    lang: ['db2', 'n1ql', 'pl/sql', 'sql'],
    indent: [
      { value: '', text: '0 spaces' },
      { value: '  ', text: '2 spaces' },
      { value: '    ', text: '4 spaces' },
      { value: '      ', text: '6 spaces' },
      { value: '        ', text: '8 spaces' },
      { value: '\t', text: 'tab' },
    ],
    linesBetweenQueries: [
      { value: 0, text: '0 lines between queries' },
      { value: 1, text: '1 lines between queries' },
      { value: 2, text: '2 lines between queries' },
      { value: 3, text: '3 lines between queries' },
      { value: 4, text: '4 lines between queries' },
    ],
  });
  let formatterConfig = {
    indent: '    ',
    lang: 'sql',
    linesBetweenQueries: 2,
    uppercase: true,
    keywordPosition: "standard", //| "tenSpaceLeft" | "tenSpaceRight"
    // newline: {
    //  "mode": "always" | "itemCount" | "lineWidth" | "hybrid" | "never",
    //  "itemCount"?: number // only used if newline.mode is itemCount or hybrid
    // },
    // breakBeforeBooleanOperator: bool,
    // aliasAs": "always" | "select" | "never",
    // tabulateAlias: boolean,
    commaPosition: "after", // "before" | "after" | "tabular",
    parenOptions: {
      openParenNewline: false,
      closeParenNewline: false,
    },
    // lineWidth: number,
    denseOperators: true,
    semicolonNewline: true,
  };
  let aceOptions = Object.freeze({
    lang: ['pgsql', 'mysql', 'sql', 'sqlserver'],
  });
  let aceConfig = {
    lang: 'pgsql',
  };

  function editorInit () {
    // @todo
  }
  function pushHistory () {
    if (history[0] !== value) {
      if (history.unshift(value) > maxHistory) {
        history.length = maxHistory
      }
      history = history;
    }
  }
  function minify () {
    pushHistory()
    value = value || ''
    value = value.replace(whitespaceRegEx, ' ')
    value = value.replace(leftparenthesesRegEx, '(')
    value = value.replace(rightparenthesesRegEx, ')')
    value = value + '\n'
  }
  function clear () {
    pushHistory()
    value = ''
    value = value;
  }
  function format () {
    pushHistory()
    try {
      err = null
      const options = {
        ...formatterConfig,
      }
      value = prettierSql.format(value, options) + '\n'
      value = value
    } catch (err) {
      // eslint-disable-next-line
      console.error(err)
      // eslint-disable-next-line
      err = new String(err)
    }
  }
  function undo () {
    value = history.shift()
    history = history;
    value = value;
  }
  function onEditorChange (newValue) {
    value = newValue.detail;
  }
</script>
