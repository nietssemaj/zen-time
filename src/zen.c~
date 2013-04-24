/*

  Zen Watch

  Based off Segment_Six

 */

#include "pebble_os.h"
#include "pebble_app.h"
#include "pebble_fonts.h"

#define MY_UUID { 0xAB, 0x13, 0x19, 0x35, 0x0F, 0x7D, 0x41, 0xCD, 0xAA, 0x9F, 0x00, 0x63, 0x0A, 0x50, 0x35, 0xE2 }

PBL_APP_INFO(MY_UUID, 
	"Zen Time", "neitssemaj",
	1, 0,
	RESOURCE_ID_IMAGE_MENU_ICON,
	APP_INFO_WATCH_FACE);

Window window;

Layer time_display_layer;
TextLayer text_date_layer;
RotBmpPairContainer zen_image_container;

// Longest Abbr Day, Month Date
char date_text[] = "Thur, Sept 00";
const char *current_date_text;

const GPathInfo MINUTE_SEGMENT_PATH_POINTS = {
  3,
  (GPoint []) {
    {0, 0},
    {-7, -72}, 
    {7,  -72},
  }
};

GPath minute_segment_path;

const GPathInfo HOUR_SEGMENT_PATH_POINTS_ONE = {
  3,
  (GPoint []) {
    {0, 0},
    {-9, -72}, 
    {-3,  -72},
  }
};

GPath hour_segment_path_one;

const GPathInfo HOUR_SEGMENT_PATH_POINTS_TWO = {
  3,
  (GPoint []) {
    {0, 0},
    {3, -72}, 
    {9,  -72},
  }
};

GPath hour_segment_path_two;

void time_display_layer_update_callback(Layer *me, GContext* ctx) {

  PblTm t;
  unsigned int angle;

  get_time(&t);

  GPoint center = grect_center_point(&me->frame);

  graphics_context_set_fill_color(ctx, GColorBlack);
  graphics_fill_circle(ctx, center, 72);

  // Draw hour hand 
  if (clock_is_24h_style()) {
    angle = (t.tm_hour * 15) + (t.tm_min / 4);
  } else { 
    angle = (( t.tm_hour % 12 ) * 30) + (t.tm_min / 2);
  }

  angle = angle - (angle % 6);

  graphics_context_set_fill_color(ctx, GColorWhite);

  gpath_rotate_to(&hour_segment_path_one, (TRIG_MAX_ANGLE / 360) * angle);
  gpath_draw_filled(ctx, &hour_segment_path_one);
  gpath_rotate_to(&hour_segment_path_two, (TRIG_MAX_ANGLE / 360) * angle);
  gpath_draw_filled(ctx, &hour_segment_path_two);

  // Draw minute hand
  angle = t.tm_min * 6;

  graphics_context_set_fill_color(ctx, GColorWhite);

  gpath_rotate_to(&minute_segment_path, (TRIG_MAX_ANGLE / 360) * angle);
  gpath_draw_filled(ctx, &minute_segment_path);

  // White circle around black center
  graphics_context_set_fill_color(ctx, GColorWhite); 
  graphics_fill_circle(ctx, center, 59);
  graphics_context_set_fill_color(ctx, GColorBlack); 
  graphics_fill_circle(ctx, center, 54);
}

void handle_minute_tick(AppContextRef ctx, PebbleTickEvent *t) {

  (void)t;
  (void)ctx;

  layer_mark_dirty(&time_display_layer);

  // Setup current date
  string_format_time(date_text, sizeof(date_text), "%a, %b %e", t->tick_time);
  current_date_text = text_layer_get_text(&text_date_layer);
  // Check to see if the date needs to be changed and change it if it does

  if (strcmp(current_date_text, date_text)) {
    text_layer_set_text(&text_date_layer, date_text);
  }
}

void handle_deinit(AppContextRef ctx) {
  (void)ctx;

  rotbmp_pair_deinit_container(&zen_image_container);
}

void handle_init(AppContextRef ctx) {
  (void)ctx;

  window_init(&window, "Zen Time");
  window_stack_push(&window, true);
  window_set_background_color(&window, GColorBlack);

  resource_init_current_app(&APP_RESOURCES);

  // Init the layer for the hands display
  layer_init(&time_display_layer, window.layer.frame);
  layer_set_frame(&time_display_layer, GRect(0, 0, 144, 144));
  time_display_layer.update_proc = &time_display_layer_update_callback;
  layer_add_child(&window.layer, &time_display_layer);

  // Init the minute segment path
  gpath_init(&minute_segment_path, &MINUTE_SEGMENT_PATH_POINTS);
  gpath_move_to(&minute_segment_path, grect_center_point(&time_display_layer.frame));

  // Init the hour segment paths
  gpath_init(&hour_segment_path_one, &HOUR_SEGMENT_PATH_POINTS_ONE);
  gpath_move_to(&hour_segment_path_one, grect_center_point(&time_display_layer.frame));
  gpath_init(&hour_segment_path_two, &HOUR_SEGMENT_PATH_POINTS_TWO);
  gpath_move_to(&hour_segment_path_two, grect_center_point(&time_display_layer.frame));

  // Init the yin yang image
  rotbmp_pair_init_container(RESOURCE_ID_IMAGE_YIN_YANG_WHITE, RESOURCE_ID_IMAGE_YIN_YANG_BLACK, &zen_image_container);
  layer_set_frame(&zen_image_container.layer.layer, GRect(3, 3, 144, 144));
  layer_add_child(&window.layer, &zen_image_container.layer.layer);


  // Init the layer for the Day, Month Year display
  text_layer_init(&text_date_layer, window.layer.frame);
  text_layer_set_text_color(&text_date_layer, GColorWhite);
  text_layer_set_background_color(&text_date_layer, GColorClear);
  text_layer_set_text_alignment(&text_date_layer, GTextAlignmentCenter);
  layer_set_frame(&text_date_layer.layer, GRect(0, 150, 144, 168-150));
  text_layer_set_font(&text_date_layer, fonts_load_custom_font(resource_get_handle(RESOURCE_ID_FONT_MONOSPACE_TYPEWRITER_18)));
  layer_add_child(&window.layer, &text_date_layer.layer);


/*
  #if SHOW_TEXT_TIME

  text_layer_init(&text_time_layer, window.layer.frame);
  text_layer_set_text_color(&text_time_layer, GColorWhite);
  text_layer_set_background_color(&text_time_layer, GColorClear);

  #if SHOW_TEXT_DATE
  layer_set_frame(&text_time_layer.layer, GRect(47, 57, 144-47, 168-57));
  #else
  layer_set_frame(&text_time_layer.layer, GRect(47, 70, 144-47, 168-70));
  #endif

  text_layer_set_font(&text_time_layer, fonts_load_custom_font(resource_get_handle(RESOURCE_ID_FONT_ROBOTO_CONDENSED_21)));
  layer_add_child(&window.layer, &text_time_layer.layer);

  #endif

  #if SHOW_TEXT_DATE

  text_layer_init(&text_date_layer, window.layer.frame);
  text_layer_set_text_color(&text_date_layer, GColorWhite);
  text_layer_set_background_color(&text_date_layer, GColorClear);
    #if SHOW_TEXT_TIME
    layer_set_frame(&text_date_layer.layer, GRect(44, 80, 144-44, 168-80));
    #else
    layer_set_frame(&text_date_layer.layer, GRect(44, 70, 144-44, 168-70));
    #endif
  text_layer_set_font(&text_date_layer, fonts_load_custom_font(resource_get_handle(RESOURCE_ID_FONT_ROBOTO_CONDENSED_21)));
  layer_add_child(&window.layer, &text_date_layer.layer);

  #endif
*/
}


void pbl_main(void *params) {
  PebbleAppHandlers handlers = {
    .init_handler = &handle_init,
    .deinit_handler = &handle_deinit,

    // Handle time updates
    .tick_info = {
      .tick_handler = &handle_minute_tick,
      .tick_units = MINUTE_UNIT
    }

  };
  app_event_loop(params, &handlers);
}
