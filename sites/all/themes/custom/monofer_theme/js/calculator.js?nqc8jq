(function ($) {
  var patientCount;
  var storedRows;
  var shouldClearOnFocus = false;
  var app;
  Drupal.behaviors.calculator = {
    attach: function (context, settings) {
      // if ie browser version is older than ie7
      if($.browser.msie && parseInt($.browser.version, 10) <= 7) {
        $(".overlay-bg", context).show();
        $("#popup-oldbrowser", context).show();
        $("#start-popup", context).hide();
      }

      //Handle printing
      $(".print", context).click(function(event) {
        var type = $(this).hasClass('individual') ? 'individual' : 'simplified';
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Print', type]);
        if(settings.monofer.app) {
          window.location = 'print://now';
        } else if(!settings.monofer.printSupported) {
          Drupal.behaviors.calculator.showPopup(Drupal.t('To print, please use your browser\'s built-in print functionality, usually found under settings', {}, {context:'Printing'}), context);
        } else {
          window.print();
        }
      });

      //Handle/init settings if php has not been run (if offline) assume mobile
      if(!window.location.origin) {
        window.location.origin = window.location.protocol+"//"+window.location.host;
      }

      app = settings.monofer.app;

      //Handle disclaimer popup
      var acceptPopup = $.cookie('accept-popup');
      var noPrompt = location.href.indexOf('noprompt')!=-1;
      if(app || noPrompt || acceptPopup!=undefined) {
        $("#start-popup", context).hide();
        $(".overlay-bg", context).hide();

        //Show video for noprompt/app users on first visit
        // if(acceptPopup==undefined && (settings.monofer.app || noPrompt)) {
        //   Drupal.behaviors.calculator.setVideoSrc(context, settings);
        //   $("body", context).addClass('mobile-overlay');
        //   $("#video-prompt", context).show();
        // } else {
          var player = videojs('instruction-video');
          player.dispose();
        // }

        //Extend life of accept-popup cookie
        $.cookie('accept-popup', 'YES', { expires: Drupal.behaviors.calculator.dateWithDaysFromNow(365), path: '/' });
      } else {
        $("body", context).addClass('mobile-prompt');
      }

      $("#video-close", context).click(function(event) {
        var player = videojs('instruction-video');
        player.pause();
        player.dispose();
        $("#video-prompt", context).hide();

        //Show download-app popup if applicable
        if(Drupal.behaviors.calculator.isTouchDevice() && !app) {
          var downloadPopup = $("#popup-downloadapp", context);
          $("#close-downloadapp", context).click(function(event) {
            $(".overlay-bg", context).hide();
            $("body", context).removeClass('mobile-overlay');
            downloadPopup.hide();
          });
          $(".appgo", context).click(function(event) {
            $(".overlay-bg", context).hide();
            $("body", context).removeClass('mobile-overlay');
            downloadPopup.hide();
          });
          downloadPopup.show();
        } else {
          $(".overlay-bg", context).hide();
          $("body", context).removeClass('mobile-overlay');
        }
      });

      $("#start-ok", context).click(function(event) {
        $("#start-popup", context).hide();

        $(".overlay-bg", context).hide();
        $("body", context).removeClass('mobile-prompt');
        $.cookie('accept-popup', 'YES', { expires: Drupal.behaviors.calculator.dateWithDaysFromNow(365), path: '/' });

        //Handle instruction video
        Drupal.behaviors.calculator.setVideoSrc(context, settings);
        $("body", context).addClass('mobile-overlay');
        $("#video-prompt", context).show();
      });

      //Handle tabs
      var activeIndex = 0;
      if(window.location.href.indexOf("simplified-page")!=-1) {
        activeIndex = 1;
      } else if(window.location.href.indexOf("individual-page")!=-1) {
        activeIndex = 0;
      } else {
        var activeTab = $.cookie('active-tab');
        activeIndex = activeTab==undefined ? 0 : activeTab;
      }
      var page = activeIndex == 0 ? 'individual-page' : 'simplified-page';
      Drupal.behaviors.calculator.gaPush(['_trackPageview', '/?' + page]);
      $(".tabs li.last", context).click(function(event) {
       $("body", context).addClass('simplified-isactive');
      });
      $('.tabs li.first', context).click(function(event) {
       $("body", context).removeClass('simplified-isactive');
      });
      $(".content", context).tabs({
        create: function( event, ui ) {
          if ($(".tabs li.last.ui-state-active", context)[0]){
             $("body").addClass('simplified-isactive');
          }
        }, active: activeIndex,
        activate: function(event, ui) {
            $.cookie('active-tab', ui.newTab.index(), { expires: Drupal.behaviors.calculator.dateWithDaysFromNow(365), path: '/' });
            var page = ui.newTab.index() == 0 ? 'individual-page' : 'simplified-page';
            Drupal.behaviors.calculator.gaPush(['_trackPageview', '/?' + page]);
        }
      });

      //Handle tooltips/info popups in general
      $('.tooltip', context).each(function(index, el) {
        var content = '<h3>' + $(this).data("title") + '</h3><p>' + $(this).data("content") + '</p>';
        $(this).tooltipsy({
          className: 'tooltipsy standard',
          content: content,
          show:function(e, $el) {
            if(Drupal.behaviors.calculator.isTouchDevice()) { //Use popup for mobile
              Drupal.behaviors.calculator.showPopup($el.find('.tooltipsy').html(), context);
            } else {
              $el.show();
            }
          }
        });
      });
      $(".close-button", context).click(function(event) {
        $("body").removeClass('mobile-overlay');
        $("#popup").hide();
        $('body').unbind('touchmove');
      });

      //Handle scrolling
      $(".totop", context).click(function() {
        $("html, body", context).animate({ scrollTop: 0 }, "slow");
      });

      //Handle accordion
      $("#acc-wrap", context).accordion({
        header: ".acc-header",
        collapsible: true,
        active: false
      });

      //Handle input types
      $(".integer-only", context).on('keydown',function(e) {
        var allowedKeys = [8, 9, 13, 37, 38, 39, 40, 46, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 110, 188, 190];
        if (allowedKeys.indexOf(e.keyCode)==-1) {
          return(false); //stop processing
        }
      });
      //Handle focus
      $(".integer-only", context).on('focus',function(e) {
        if(shouldClearOnFocus) { //Clear input
          shouldClearOnFocus = false;
          $("input[name='irons']", context).val('');
          $("input[name='targethb']", context).val('');
          $("input[name='actualhb']", context).val('');
          $("input[name='idealbw']", context).val('');
          $("input[name='patient-id']", context).val('');
          //Patient Iron Need
          $('#iron-need-result', context).html('–');
          //Recommendation
          $(".iron-recommendation-result", context).hide();
          $(".empty-result", context).show();
          $(".recommendation .result-image", context).hide();
          $(".info-followup", context).hide();
          //Monofer Dose
          $(".monofer-dose-result", context).html('–');
        }
      });

      //Submit handling
      patientCount = 0;
      $(".patient-submit", context).click(function(event) {
        event.preventDefault();
        if(storedRows.length>0) {
          var patientID = $("input[name='patient-id']", context).val();
          Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Patient ID', 'Individual', patientID]);
          $('.result-table tr:eq(1) .patient', context).html(patientID);
          $(".mobile-patient", context).first().html(patientID);
          storedRows[0].id = patientID;
          Drupal.behaviors.calculator.updateCookie();
        }
      });
      $(".calculate-btn", context).click(function(event) {
        event.preventDefault(); //Avoid submit/refresh
        Drupal.behaviors.calculator.calculate(context, settings);
      });

      //Handle deletion of rows
      $('.delete', context).live('click', function () {
        var index = $(".delete", context).index(this);
        Drupal.behaviors.calculator.deleteRow(index, context);
      });
      $(".mobile-delete", context).live('click', function() {
        var index = $(".mobile-delete", context).index(this);
        Drupal.behaviors.calculator.deleteRow(index, context);
      });

      $("select[name='units']", context).bind($.browser.msie ? 'click' : 'change', function(event) {
        var unitVal = $(this).val();
        $("select[name='units-simplified']", context).val(unitVal);
        $.cookie('unit', unitVal, { expires: Drupal.behaviors.calculator.dateWithDaysFromNow(365), path: '/' });
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Unit Selection', 'Individual', unitVal]);
      });

      //Handle custom unit selection
      $(".active-unit", context).click(function(event) {
        $(this).next().show();
      });
      $(".unit-select.individual .unit-list li", context).click(function(event) {
        var unitContainer = $(this).parent().prev().find('.unit-value');
        unitContainer.html($(this).text());
        var data = $(this).data('val');
        unitContainer.data('val', data);

        var simpleContainer = $(".unit-select.simplified .unit-value", context);
        simpleContainer.html($(this).text());
        simpleContainer.data('val', data);

        $(this).parent().hide();
        $.cookie('unit', data, { expires: Drupal.behaviors.calculator.dateWithDaysFromNow(365), path: '/' });
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Unit Selection', 'Individual', data]);
      });

      var unit = $.cookie('unit');
      if(unit!==undefined) {
        if(settings.monofer.mobile) { //Use native selects
          $("select[name='units']", context).val(unit);
          $("select[name='units-simplified']", context).val(unit);
          var unitText = $("select[name='units-simplified'] option:selected", context).text();
          Drupal.behaviors.calculator.updateSimplifiedUnits(unit, unitText, settings, context);
        } else {
          var unitText = $(".unit-select.individual ul").find("[data-val='" + unit + "']").html();
          var commonContainer = $(".unit-select .active-unit .unit-value", context);
          commonContainer.html(unitText);
          commonContainer.data('val', unit);
        }
      }
      //And unselection
      $(document).mouseup(function (e) {
        var container = $(".unit-select");
        var child = $(".unit-list");
        if (!container.is(e.target) // if the target of the click isn't the container...
            && container.has(e.target).length === 0) // ... nor a descendant of the container
        {
          child.hide();
        }
      });

      //Simplified table
      $(".simplified-table td.clickable", context).click(function(event) {
        var index = $(this).data("index");
        var simplifiedText = $(".simplified-table tr.simplified-content:eq(" + index + ")", context);
        var action;
        if($(this).hasClass('active')) {
          $(this).removeClass('active');
          simplifiedText.hide();
          action = 'close';
        } else {
          $("td.active", context).removeClass('active');
          $('tr.simplified-content', context).hide();
          $(this).addClass('active');
          simplifiedText.show();
          action = 'open';
        }
        var simplifiedLabel;
        if(index=="0") {
          simplifiedLabel = "1000mg >= 10";
        } else if(index=="1") {
          simplifiedLabel = "1500mg >= 10";
        } else if(index=="2") {
          simplifiedLabel = "1500mg < 10";
        } else {
          simplifiedLabel = "2000mg < 10";
        }
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Patient Iron Need', action, simplifiedLabel]);
      });

      $("select[name='units-simplified']", context).bind($.browser.msie ? 'click' : 'change', function(event) {
        var unitVal = $(this).val();
        $("select[name='units']", context).val(unitVal);
        var unitText = $(this).find("option:selected").text();
        Drupal.behaviors.calculator.updateSimplifiedUnits(unitVal, unitText, settings, context);
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Unit Selection', 'Simplified', unitVal]);
      });

      $(".unit-select.simplified .unit-list li", context).click(function(event) {
        var unitContainer = $(this).parent().prev().find('.unit-value');
        var unitVal = $(this).data('val');
        var unitText = $(this).text();
        unitContainer.html(unitText);
        unitContainer.data('val', unitVal);

        var individualContainer = $(".unit-select.individual .unit-value", context);
        individualContainer.html(unitText);
        individualContainer.data('val', unitVal);

        Drupal.behaviors.calculator.updateSimplifiedUnits(unitVal, unitText, settings, context);
        $(this).parent().hide();
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Unit Selection', 'Simplified', unitVal]);
      });
      $('#mobile-info-tooltip', context).tooltipsy({
        className: 'tooltipsy',
        show:function(e, $el) {
          Drupal.behaviors.calculator.showPopup($("#mobile-info-tooltip-content", context).html(), context, 'mobile-tooltipsy');
        }
      });

      var infotextTable = '<h3>' + Drupal.t('Recommendation') + '</h3>';
      infotextTable += '<p>' + Drupal.t('Monofer can be administered up to 20 mg/kg.') + '</p>';
      infotextTable += '<div class="result-image one-visit"></div>';
      infotextTable += '<p>' + Drupal.t('Full iron correction can be achieved in just one visit') + '</p>';
      infotextTable += '<div class="result-image follow-up"></div>';
      infotextTable += '<p>' + Drupal.t('If the patient’s need for iron exceeds 20 mg/kg body weight, the dose must be divided and administered at intervals of at least one week. It is recommended to administer maximum dose at the first visit.') + '</p>';
      $('#info-followup-table', context).tooltipsy({
        content: infotextTable,
        className: 'tooltipsy info-wide',
        show: function(e, $el) {
          if(Drupal.behaviors.calculator.isTouchDevice()) {
            Drupal.behaviors.calculator.showPopup(infotextTable, context);
          } else {
            $el.show();
          }
        }
      });
      $('.first-dose-tooltip', context).live('click', function() {
        Drupal.behaviors.calculator.showPopup(infotextTable, context);
      });

      //Import stored data from cookie
      storedRows = [];
      if(settings.monofer.app || document.referrer.indexOf(window.location.origin)!=-1) { //Only use stored data if on same domain or an app
        var count = $.cookie('counter');
        if(count!=undefined) {
          patientCount = parseInt(count);
        }
        var rows = $.cookie('rows');
        if(rows!=undefined) {
          rows = JSON.parse(rows);
          var row;
          for (var i = 0; i < rows.length; i++) {
            row = rows[i];
            Drupal.behaviors.calculator.addRow(context,row.id,row.ironNeed,row.dose,row.firstDose,row.stores,row.target,row.actual,row.ideal,row.visitClass);
          }
          if(rows.length>0 && row!=undefined) { //Add info to boxes on recommendation etc.
            //Patient iron need
            $('#iron-need-result', context).html(row.ironNeed + ' mg');

            //Recommendation
            var infotext = '<h3>' + Drupal.t('Recommendation', {}, {context:'Recommendation tooltip'}) + '</h3>' + '<p>' + Drupal.t('Monofer can be administered up to 20 mg/kg.', {}, {context:'Recommendation tooltip'}) + '</p>';
            if(row.visitClass=="one-visit") {
              recommendation = Drupal.t('Full iron correction can be achieved in just one visit', {}, {context:'Recommendation box'});
              $(".info-followup", context).show();
              infotext += '<div class="result-image one-visit"></div>' + '<p>' + Drupal.t('Full iron correction can be achieved in just one visit.', {}, {context:'Recommendation tooltip'}) + '</p>';
              Drupal.behaviors.calculator.updateInfoFollowup(true, infotext, recommendation, row.visitClass, context);
            } else {
              recommendation = Drupal.t('Maximum dose at first visit', {}, {context:'Recommendation box'});
              $(".info-followup", context).show();
              infotext += '<div class="result-image follow-up"></div>' + '<p>' + Drupal.t('If the patient’s need for iron exceeds 20 mg/kg body weight, the dose must be divided and administered at intervals of at least one week. It is recommended to administer maximum dose at the first visit.', {}, {context:'Recommendation tooltip'}) + '</p>';
              recommendation += '<span class="bold">' + Drupal.t(' @d&nbsp;ml', {'@d':row.firstDose}, {context:'Recommendation dose'}) + '</span>';
              Drupal.behaviors.calculator.updateInfoFollowup(false, infotext, recommendation, row.visitClass, context);
            }
            //Monofer Dose
            $('.monofer-dose-result', context).html(row.dose + ' ml');
          }
        }
      } else {
        $.removeCookie('counter');
        $.removeCookie('rows');
      }
    },
    calculate: function(context, settings) {
      //Remove old errors if any
      $('input.error', context).removeClass('error');

      //Verify input values
      var error = false;
      var ironStores = $("input[name='irons']", context).val();
      ironStores = Drupal.behaviors.calculator.parseNumber(ironStores);
      if(!Drupal.behaviors.calculator.isNumber(ironStores)) {
        error = true;
        $("input[name='irons']", context).addClass('error');
      } else {
        ironStores = parseFloat(ironStores);
      }
      var target = $("input[name='targethb']", context).val();
      target = Drupal.behaviors.calculator.parseNumber(target);
      if(!Drupal.behaviors.calculator.isNumber(target)) {
        error = true;
        $("input[name='targethb']", context).addClass('error');
      } else {
        target = parseFloat(target);
      }
      var actual = $("input[name='actualhb']", context).val();
      actual = Drupal.behaviors.calculator.parseNumber(actual);
      if(!Drupal.behaviors.calculator.isNumber(actual)) {
        error = true;
        $("input[name='actualhb']", context).addClass('error');
      } else {
        actual = parseFloat(actual);
      }
      var ideal = $("input[name='idealbw']", context).val();
      ideal = Drupal.behaviors.calculator.parseNumber(ideal);
      if(!Drupal.behaviors.calculator.isNumber(ideal)) {
        error = true;
        $("input[name='idealbw']", context).addClass('error');
      } else {
        ideal = parseFloat(ideal);
      }

      if(!error) { //Verify that
        if(actual>=target) {
          error = true;
          if(Drupal.behaviors.calculator.isTouchDevice()) { //Use popup for mobile
            Drupal.behaviors.calculator.showPopup($(".calc-row.actual-hb .tooltip", context).data('content'), context);
          } else {
            $(".calc-row.actual-hb .tooltip", context).data('tooltipsy').show();
            setTimeout(function() { //timeout and hide popup again
                  $(".calc-row.actual-hb .tooltip", context).data('tooltipsy').hide();
            }, 3000);
          }
        }
      }

      if(!error) {
        var unit = $("select[name='units'] option:selected", context).val();
        if(unit==undefined) { //Use the custom select's value
          unit = $(".unit-select.individual .unit-value").data('val');
        }
        var unitScale = unit=='g-dl' ? 1 : unit=='g-l' ? 0.1 : 1.61; //Scale Hb
        var totalIronNeed = ideal * (target*unitScale-actual*unitScale) * 2.4 + ironStores;
        var roundedIronNeed = Math.round(totalIronNeed/100) * 100; //To nearest 100
        var dose = roundedIronNeed/ideal;
        var roundedDose = dose.toFixed(2);

        //Patient iron need
        $('#iron-need-result', context).html(roundedIronNeed + ' mg');
        var recommendation = '';
        var visitClass;
        var infotext = '<h3>' + Drupal.t('Recommendation', {}, {context:'Recommendation tooltip'}) + '</h3>' + '<p>' + Drupal.t('Monofer can be administered up to 20 mg/kg.', {}, {context:'Recommendation tooltip'}) + '</p>';
        var firstDose;
        if(roundedDose<=20) {
          recommendation = Drupal.t('Full iron correction can be achieved in just one visit', {}, {context:'Recommendation box'});
          visitClass = "one-visit";
          $(".info-followup", context).show();
          infotext += '<div class="result-image one-visit"></div>' + '<p>' + Drupal.t('Full iron correction can be achieved in just one visit.', {}, {context:'Recommendation tooltip'}) + '</p>';
          firstDose = roundedIronNeed/100;
          Drupal.behaviors.calculator.updateInfoFollowup(true, infotext, recommendation, visitClass, context);
        } else {
          recommendation = Drupal.t('Dose must be split. Maximum dose at first visit', {}, {context:'Recommendation box'});
          visitClass = "follow-up";
          $(".info-followup", context).show();
          infotext += '<div class="result-image follow-up"></div>' + '<p>' + Drupal.t('If the patient’s need for iron exceeds 20 mg/kg body weight, the dose must be divided and administered at intervals of at least one week. It is recommended to administer maximum dose at the first visit.', {}, {context:'Recommendation tooltip'}) + '</p>';
          // firstDose = Math.round(ideal*20/100);
          firstDose = Math.floor(ideal*20/100);
          recommendation += '<span class="bold">' + Drupal.t(' @d&nbsp;ml', {'@d':firstDose}, {context:'Recommendation dose'}) + '</span>';
          Drupal.behaviors.calculator.updateInfoFollowup(false, infotext, recommendation, visitClass, context);
        }

        patientCount ++;
        var patientID = $("input[name='patient-id']", context).val();
        if(patientID=="") {
          patientID = Drupal.t("ID @id", {'@id':patientCount}, {context:'Patient ID'});
        }

        //Monofer Dose
        roundedDose = roundedIronNeed/100;
        $('.monofer-dose-result', context).html(roundedDose + ' ml');

        Drupal.behaviors.calculator.addRow(context, patientID, roundedIronNeed, roundedDose, firstDose, ironStores, target, actual, ideal, visitClass);

        shouldClearOnFocus = true;

        if(screen.width<768) {
          $('html, body', context).animate({scrollTop:$('#result-anchor', context).offset().top}, 'slow');
        } else {
          $('html, body', context).animate({scrollTop:0}, 'slow');
        }
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Dose calculation status', 'Calculate', 'Calculation success']);
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Dose calculation input', 'Unit Selection', unit]);
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Dose calculation input', 'Iron Stores', '', ironStores]);
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Dose calculation input', 'Target Hb', '', target]);
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Dose calculation input', 'Actual Hb', '', actual]);
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Dose calculation input', 'Body weight', '', ideal]);
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Dose calculation input', 'Patient Iron Need', '', roundedIronNeed]);
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Dose calculation input', 'Monofer Dose', '', roundedDose]);
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Dose calculation input', 'Recommendation', visitClass]);
      } else {
        Drupal.behaviors.calculator.gaPush(['_trackEvent', 'Dose calculation status', 'Calculate', 'Calculation input error']);
      }
    },
    addRow: function(context, id, ironNeed, dose, firstDose, stores, target, actual, ideal, visitClass) {
      $('.result-table .first-row', context).show();
      $('.result-table .last-row', context).removeClass('last-row');
      var unit = $("select[name='units'] option:selected", context).text();
      if(unit=='') { //Use the custom select's value
        unit = $(".unit-select.individual .unit-value").text();
      }
      unit = ' ' + unit;
      var row = '<tr>' +
            '<td class="first-cell"><span class="' + visitClass + '"></span></td>' +
            // '<td class="patient">' + id + '</td>' +
            '<td class="ironneed">' + ironNeed + ' mg</td>' +
            '<td class="monoferdose">' + dose + ' ml</td>' +
            '<td class="firstdose">' + firstDose + ' ml</td>' +
            '<td class="stores">' + stores + ' mg</td>' +
            '<td>' + target + unit + '</td>' +
            '<td>' + actual + unit + '</td>' +
            '<td>' + ideal + ' kg</td>' +
            '<td class="last-cell"><span class="delete"></span></td>' +
          '</tr>';
      $(".result-table tr", context).first().after(row);
      $(".result-table tr:gt(5)", context).remove();
      $(".result-table tr", context).last().addClass('last-row');

      //Mobile table
      $(".history-title", context).show();
      var mobileRow =    '<div class="mobile-row">' +
                        '<div class="acc-header"><span class="mobile-patient">' + id + '</span><span class="' + visitClass + '"></span><span class="mobile-delete"></span><span class="arrow"></span></div>' +
                        '<div class="acc-content">' +
                          '<div class="data-row ironneed"><label>' + Drupal.t('Patient iron need', {}, {'context':'Mobile row'}) + '</label><span class="data">' + ironNeed + ' mg</span></div>' +
                          '<div class="data-row monoferdose"><label>' + Drupal.t('Monofer dose', {}, {context:'Mobile row'}) + '</label><span class="data">' + dose + ' ml</span></div>' +
                          '<div class="data-row firstdose"><label>' + Drupal.t('First dose', {}, {context:'Mobile row'}) + '</label><span class="data">' + firstDose + ' ml</span><span class="first-dose-tooltip"></span></div>' +
                          '<div class="data-row stores"><label>' + Drupal.t('Iron stores', {}, {context:'Mobile row'}) + '</label><span class="data">' + stores + ' mg</span></div>' +
                          '<div class="data-row"><label>' + Drupal.t('Target Hb', {}, {context:'Mobile row'}) + '</label><span class="data">' + target + unit + '</span></div>' +
                          '<div class="data-row"><label>' + Drupal.t('Actual Hb', {}, {context:'Mobile row'}) + '</label><span class="data">' + actual + unit + '</span></div>' +
                          '<div class="data-row"><label>' + Drupal.t('Ideal bw', {}, {context:'Mobile row'}) + '</label><span class="data">' + ideal + ' kg</span></div>' +
                        '</div>' +
                      '</div>';
      $(".mobile-result-table .history-title", context).after(mobileRow);
      $(".mobile-row:gt(4)", context).remove();
      $("#acc-wrap", context).accordion('destroy').accordion({
        header: ".acc-header",
        collapsible: true,
        active: false
      });

      //Maintain storedRows
      storedRows.push({
        'id':id,
        'ironNeed':ironNeed,
        'dose':dose,
        'firstDose':firstDose,
        'stores':stores,
        'target':target,
        'actual':actual,
        'ideal':ideal,
        'visitClass':visitClass
      });
      if(storedRows.length>5) {
        storedRows.splice(0, storedRows.length-5);
      }
      Drupal.behaviors.calculator.updateCookie();
    },
    deleteRow: function(index, context) {
      var row = $(".delete", context).eq(index).closest('tr').remove();
      var rows = $(".result-table tr", context);
      if(rows.length>1) {
        rows.last().addClass('last-row');
      }
      //Delete mobile row
      $(".mobile-row", context)[index].remove();

      storedRows.splice(storedRows.length-index-1, 1);
      Drupal.behaviors.calculator.updateCookie();
    },
    showPopup: function(content, context, customClass) {
      $("#popup-message", context).removeClass();
      if(customClass!==undefined) {
        $("#popup-message", context).addClass(customClass);
      }
      $("#popup-message", context).html(content);
      $("#popup", context).show();
      $("body").addClass('mobile-overlay');
      $('body').bind('touchmove', function(e){e.preventDefault()});
    },
    parseNumber: function(n) {
      if(n!==undefined) {
        n = n.replace(",", ".");
      }
      return n;
    },
    isNumber: function(n) {
      return !isNaN(parseFloat(n)) && isFinite(n);
    },
    updateCookie: function() {
      $.cookie('rows', JSON.stringify(storedRows));
      $.cookie('counter', '' + patientCount);
      $.cookie('counter', Number);
    },
    isTouchDevice: function() {
      //DEBUG
      //return true;
      if(Drupal.behaviors.calculator.isIE()) {
        return false;
      }
      return 'ontouchstart' in window // works on most browsers
          || 'onmsgesturechange' in window; // works on ie10
    },
    updateSimplifiedUnits: function(unitVal, unitText, settings, context) {
      $("#simple-unit-header").html(Drupal.t("Hb (@unit)", {'@unit':unitText}, {context:'Simple unit'}));
      var unitScale = unitVal=='g-dl' ? 1 : unitVal=='g-l' ? 10 : 0.62; //Scale Hb
      var threshold = (10*unitScale);
      $("#simplified-hb-1", context).html('≥ ' + threshold);
      $("#simplified-hb-2", context).html('< ' + threshold);
      $.cookie('unit', unitVal, { expires: Drupal.behaviors.calculator.dateWithDaysFromNow(365), path: '/' });
    },
    updateInfoFollowup:function(singleVisit, infotext, recommendation, visitClass, context) {
      if($('.recommendation-tooltip.info-followup', context).data('tooltipsy')!==undefined) {
        $('.tooltipsy.info-wide.info-left', context).parent().remove();
      }
      $('.recommendation-tooltip.info-followup', context).tooltipsy({
         content: infotext,
         className: 'tooltipsy info-wide info-left ' + visitClass + '-tooltip',
         offset: [-1, 0], //Show to the left of the info icon
         show:function(e, $el) {
          if(Drupal.behaviors.calculator.isTouchDevice()) { //Use popup for mobile
            Drupal.behaviors.calculator.showPopup(infotext, context);
          } else {
            $el.show();
          }
         }
      });
      $('.empty-result', context).hide();
      $('.result-box .result-image', context).attr("class",'result-image ' + visitClass);
      $('.result-box .result-image', context).show();

      $('.iron-recommendation-result', context).html(recommendation);
      $('.iron-recommendation-result', context).show(); //Show if it was hidden
    },
    isIE: function() {
      var myNav = navigator.userAgent.toLowerCase();
      // return (myNav.indexOf('msie') != -1) ? parseInt(myNav.split('msie')[1]) : false;
      return (myNav.indexOf('msie') != -1) ? true : false;
    },
    gaPush: function(vars) {
      if(app) {
        var customUrl = 'analytics://' + vars;
        Drupal.behaviors.calculator.execute(customUrl);
      } else {
        try{
          _gaq.push(vars);
        } catch(err) {
        }
      }
    },
    execute:function(url) {
      var iframe = document.createElement("IFRAME");
      iframe.setAttribute("src", url);
      document.documentElement.appendChild(iframe);
      iframe.parentNode.removeChild(iframe);
      iframe = null;
    },
    dateWithDaysFromNow:function(days) {
      var date = new Date();
      date.setTime(date.getTime() + (days*24*60*60*1000));
      return date;
    },
    setVideoSrc:function(context, settings) {
      var src;
      if(window.screen.width > 760) {
        src = settings.monofer.ipad_video;
      } else {
        src = settings.monofer.mobile_video;
      }
      var player = videojs('instruction-video');
      player.src({ type: "video/mp4", src: src });
    }
  };
})(jQuery);
