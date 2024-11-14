# PayNow Payments Cards, Ecocash, One Money

## Create an Account

## Get Intergration Keys

## Routes

```bash

Route::post('/paynow/initiate', [PaynowController::class, 'initiatePayment']);
Route::post('/paynow/initiate-mobile', [PaynowController::class, 'initiateMobilePayment']);
Route::post('/paynow/status', [PaynowController::class, 'checkPaymentStatus']);

```