(function(window) {

})(window)

why window?  for gzip

(function(window, undefined) {
  
})

why undefine?  undefined can be var in old ie


function jQuery() {
  return new jQuery.fn.init()
}

jQuery.fn = jQuery.prototype

jQuery.prototype.init.prototype = jQuery.prototype

jQuery.get(-1)

jQuery.eq(-1)

expando  data()
