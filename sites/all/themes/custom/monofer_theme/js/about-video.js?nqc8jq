(function ($) {
  Drupal.behaviors.about = {
    attach: function (context, settings) {
      $(".show-video", context).click(function(event) {
        Drupal.behaviors.about.setVideoSrc(context, settings);
        $("body", context).addClass('mobile-overlay');
        // $(".overlay-bg", context).show();
        $("#video-prompt", context).show();
      });

      $("#video-close", context).click(function(event) {
        var player = videojs('instruction-video');
        player.pause();
        $("#video-prompt", context).hide();
        $(".overlay-bg", context).hide();
        $("body", context).removeClass('mobile-overlay');
      });
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
  }
})(jQuery);
