# Clear static_content cache on all instances in the prod zone every on sundays at 4am local server time

bundle common cache_maintance_file_control
{
  vars:
    "inputs" slist => {
                        "$(this.promise_dirname)/../../../../../lib/3.6/stdlib.cf",
                      };

}
body file control
{
    inputs => { @(cache_maintance_file_control.inputs) };
}
bundle agent cache_maintance
{
  vars:
    "static_files"
      slist => { ".*\.jpg" };

  classes:
    "clear_prod_static_content_caches"
      and => {
               "Sunday",
               "Hr04_Q4",
               "nginx_plus_server_zone_prod",
               "nginx_plus_cache_static_content",
             };

  files:
    clear_prod_static_content_caches::
      "/var/nginx/cache/static_content/."
        delete => tidy,
        depth_search => recurse("inf"),
        file_select => by_name( @(static_files) ),
        action => if_elapsed_day;
}
