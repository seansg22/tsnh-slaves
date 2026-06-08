```
  ┌─────────────────────────────────────────────────────────────────────────────┐
  │                         ENTRY POINTS (Existing)                             │
  └─────────────────────────────────────────────────────────────────────────────┘

    OperationTimeSettingList.vue ──── onMounted ──► getSellerInstantPickupTime()
    (EXISTING, modified)                           getShopOperatingHourConfig()  ◄─ NEW call
             │
             │  pickupTimeSlots in store?
             ├─── YES ──► InstantPreparationPickupSlotsSetting.vue  ◄─ NEW
             │
             └─── NO  ──► OperationTimeSettingItem.vue               (EXISTING, unchanged)


    TurboDeliverySetting.vue (page) ──── isInstantRolloverOrdersEnabled?
    (EXISTING, modified)                │
                                        └─► OutsideInstantOperatingHoursCapacity.vue  ◄─ NEW


    SetCapacityButtons.vue ──── updateChannelToggle error code?
    (EXISTING, modified)        │
                                ├── 901 ──► InstantOperatingHourModal              (EXISTING)
                                │
                                └── 905 ──► InstantOptWithPickupTimeSlotsModal.vue  ◄─ NEW


  ┌─────────────────────────────────────────────────────────────────────────────┐
  │              NEW COMPONENT TREE (fulfillment-settings)                      │
  └─────────────────────────────────────────────────────────────────────────────┘

    InstantPreparationPickupSlotsSetting.vue   ◄─ Orchestrator (edit/view toggle)
    ├── isEditing = false
    │     └──► InstantPreparationPickupSlotsSummary.vue  ◄─ READ from store
    │              reads: store.instantOperatingHours.pickupTimeSlots
    │              reads: store.instantOperatingHours.preparationTimeInterval
    │              reads: store.instantOperatingHours.regular
    │
    └── isEditing = true
          └──► InstantPreparationPickupSlotsSection.vue  ◄─ Edit wrapper
                   │  delegates all logic to:
                   └──► usePreparationPickupSlots.ts  ◄─ Composable (all business logic)
                            │  calls APIs:
                            ├── fetchPreparationTimeIntervalList()        ◄─ NEW API
                            ├── fetchSellerInstantPickupTime()            (EXTENDED)
                            ├── validateSellerInstantPickupTime()         ◄─ NEW API
                            └── setSellerInstantPickupTime()              (EXTENDED)

    InstantPreparationPickupSlotsSection.vue
    ├── OperatingHoursTimeSlotErrorBanner.vue  ◄─ NEW (warning notice)
    ├── PreparationTimeSelector.vue            ◄─ NEW (interval radio group)
    ├── PickupSlotDayPanel.vue × N days        ◄─ NEW (per-day panel)
    │     ├── OperationHoursTimeRangePicker    (EXISTING shared lib)
    │     └── PickupSlotButton.vue × N slots   ◄─ NEW
    │             └── CheckboxWithCheckedStyle.vue  ◄─ NEW (styled toggle)
    └── PreparationPickupGuideModal.vue        ◄─ NEW (first-time onboard)

    InstantOptWithPickupTimeSlotsModal.vue     ◄─ Modal wrapper for turbo toggle flow
    └── InstantPreparationPickupSlotsSection.vue  (reuses same edit section)


  ┌─────────────────────────────────────────────────────────────────────────────┐
  │                         DATA FLOW                                           │
  └─────────────────────────────────────────────────────────────────────────────┘

    GET /get_shop_instant_pickup_time
    (pickupTimeSlots present?)
         │
         YES ──► operationHourSetting store
                 ├── regular              (existing)
                 ├── pickupTimeSlots      ◄─ NEW
                 └── preparationTimeInterval  ◄─ NEW
                           │
                           └──► InstantPreparationPickupSlotsSetting (reads store)

    GET /get_preparation_time_interval_list  ◄─ NEW
         └──► PreparationTimeSelector (local state via composable)

    POST /validate_shop_instant_pickup_time  ◄─ NEW
         └──► per-day validationState (local to composable)
                   └──► PickupSlotDayPanel (error / isResolvable / isSuggested)

    POST /set_shop_instant_pickup_time  (EXTENDED: now sends preparationTimeInterval + pickupTimeSlots)

    GET/SET /get_shop_channel_capacity  (EXTENDED: capacitySetting now includes outsideHoursDaily)
         └──► turboDeliverySetting store
                 └── outsideInstantOperatingHoursCapacity  ◄─ NEW
                           └──► OutsideInstantOperatingHoursCapacity.vue


  ┌─────────────────────────────────────────────────────────────────────────────┐
  │              SHARED LIB CHANGES (libs/fulfillment)                          │
  └─────────────────────────────────────────────────────────────────────────────┘

    OperationHoursTimePicker.vue   ◄─ Bug fix: disabled now shows actual value
                                      (used inside PickupSlotDayPanel via
                                       OperationHoursTimeRangePicker)

    TurboDeliveryCapacityDay enum  ◄─ Added OUTSIDE_HOURS_DAILY
    UpdateChannelErrorCode enum    ◄─ Added ERROR code 905
    ShopChannelCapacitySetting     ◄─ New type extending Record with outsideHoursDaily?
```