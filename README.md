## Welcome to Guacamol

The Leaderboard and results will come available here soon...

<h3>Distribution benchmarks</h3>
<div class="guacamol-table" id="jsGrid"></div>
<h3>Goal directed benchmarks</h3>

<div class="guacamol-table" id="jsGrid2"></div>

<script type="text/javascript" src="https://code.jquery.com/jquery-2.2.4.min.js"></script>

<script type="text/javascript">

  /*
   * TODO:
   * Both JSON have been hardcoded here due to the gitlab CORS policy
   * Once this is changed you can remove table1 and table2 variables and re-enable ajax requests below
   */

  let baseUrl = "https://benevolentai.github.io/guacamol_results/guacamol/";

  let tableDistUrl = baseUrl + 'table_dist_flip.json';
  ;
  let tableGoalUrl = baseUrl + 'table_goal_flip.json';

  function linkeRenderer(value, text) {

    if(!value.hasOwnProperty("link")) return ' ';
    return `<a href='${baseUrl + value.link}' target='_blank' class='molecule-link'> ${value.score || text}`;

  }

  let createTable = function (id, data, linkText, width) {

    if (!data.length) {
      alert("Json Data is missing or not a list")
    }

    const linkData = data.splice(0, 1)[0];

    let fields = Object.keys(data[0])
      .map((a) => {

        if ((typeof data[0][a] === 'object' && data[0][a].hasOwnProperty("link"))) {
          return {
            name: a, type: "text", width: 150, align: "center",
            itemTemplate: function (value) {
              return $("<a href='" + baseUrl + (value.link) + "' target='_blank'>").addClass("molecule-link").text(value.score || "see molecules");
            },

            sorter: function(value1, value2) {
              if (value1.score == value2.score) {
                return 0
              }
              return value1.score > value2.score ? 1 : -1;
            }
          }

        }
        return {
          name: a,
          type: "text",
          align: "center",
          width: 150
        }
      });

    $(id).jsGrid({
      height: "auto",
      width: width,
      sorting: true,
      paging: true,
      fields: fields,
      data: data,
      // Custom prop
      baiLinkData: linkData,
      baiLinkText: linkText,
      onRefreshed: function(args) {
       let tds = '';
       const linkData = args.grid.baiLinkData;
       const fields =  args.grid.fields;

       for(var i = 0; i < fields.length; i++) {
         const linkObj = linkData[fields[i].name];
         tds += `<td class="jsgrid-cell jsgrid-align-center" stylex="width: 150px;">`
           + linkeRenderer(linkObj, args.grid.baiLinkText)
           + `</td>`;
       }
       const $row = $(`<tr class="jsgrid-row" >` + tds + `</tr>`);

        args.grid._content.append($row);
      }
    });
  }

  const drawTables = function(dataDist, dataGoal) {
    const screenWidth = window.innerWidth;
    const maxDistTableWidth = Object.keys(dataDist[0][0]).length*150;
    const maxGoalTableWidth = Object.keys(dataGoal[0][0]).length*150;
    const mobileBreakpoint = 640;
    const largeMobileBreakpoint = 770;
    
    let fullScreenPadding = "40px";
    if (window.innerWidth >= mobileBreakpoint && window.innerWidth < largeMobileBreakpoint) {
      fullScreenPadding = "72px"
    }
    
    if (window.innerWidth >= largeMobileBreakpoint) {
      fullScreenPadding = "96px";
    }
    
    const distTableWidth = screenWidth > maxDistTableWidth ? maxDistTableWidth : "calc(100vw - " + fullScreenPadding + ")";
    const goalTableWidth = screenWidth > maxGoalTableWidth ? maxGoalTableWidth : "calc(100vw - " + fullScreenPadding +  ")";
    createTable('#jsGrid',dataDist[0],'see molecules', distTableWidth);
    createTable('#jsGrid2',dataGoal[0], 'json data', goalTableWidth)
  };

  $(function () {
    // TODO: re-enable when CORS are enabled
    $.when($.get(tableDistUrl), $.get(tableGoalUrl))
      .done((dataDist, dataGoal) => {
        drawTables(dataDist, dataGoal);
      });


  })

</script>
