
<script>
  export default {
    mpType: 'app',
    onLaunch: function(options) {
        wx.cloud.init();
    },
    onShow: function(options) {
      // Do something when show.
      console.log('App onShow')
    },
    onHide: function() {
      // Do something when hide.
      console.log('App onHide')
    },
    onError: function(msg) {
      console.log('App onError')
      console.log(msg)
    }
  }
</script>
