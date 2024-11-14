# PayNow Payments Cards, Ecocash, One Money

## Create an Account
 on Paynow and follow the email validation steps. https://www.paynow.co.zw/Customer/Register
Login with your newly created account and setup the bank account details you would like to receive payment into.
Go to the [Other Ways To Get Paid] page. https://www.paynow.co.zw/Home/Receive

## Get Intergration Keys

## Routes

```bash

Route::post('/paynow/initiate', [PaynowController::class, 'initiatePayment']);
Route::post('/paynow/initiate-mobile', [PaynowController::class, 'initiateMobilePayment']);
Route::post('/paynow/status', [PaynowController::class, 'checkPaymentStatus']);

```

## Paynow Controller

```bash
use Paynow\Payments\Paynow;
use Illuminate\Support\Facades\Log;

class PaynowController extends Controller
{
   protected $paynow;

    public function __construct()
    {
        $this->paynow = new Paynow(
            env('PAYNOW_INTEGRATION_ID'),
            env('PAYNOW_INTEGRATION_KEY'),
            env('PAYNOW_RESULT_URL'),
            env('PAYNOW_RETURN_URL')
        );
    }

    public function initiatePayment(Request $request)
    {
        $payment = $this->paynow->createPayment($request->reference, $request->email);

        // Add items to the payment
        foreach ($request->items as $item) {
            $payment->add($item['name'], $item['price']);
        }

        // $response = $this->paynow->send($payment);
        $response = $paynow->sendMobile($payment, '0771111111', 'ecocash');

        if ($response->success()) {
            // Save pollUrl in the database for tracking status later
            $pollUrl = $response->pollUrl();
            return response()->json([
                'redirect_url' => $response->redirectUrl(),
                'poll_url' => $pollUrl
            ]);
        }

        return response()->json(['error' => 'Payment initiation failed'], 500);
    }


    public function initiateMobilePayment(Request $request)
    {
        try {
            $payment = $this->paynow->createPayment($request->reference, $request->email);

            foreach ($request->items as $item) {
                $payment->add($item['name'], $item['price']);
            }

            $response = $this->paynow->sendMobile($payment, $request->phone, $request->method);

            if ($response->success()) {
                return response()->json([
                    'poll_url' => $response->pollUrl(),
                    'instructions' => $response->instructions()
                ]);
            } else {
                // Log the full response details to see more error information
                Log::error("PayNow mobile payment failed", ['full_response' => $response]);
                return response()->json(['error' => 'Mobile payment initiation failed'], 500);
            }
        } catch (\Exception $e) {
            Log::error("Error initiating mobile payment", ['message' => $e->getMessage()]);
            return response()->json(['error' => 'An error occurred during mobile payment initiation'], 500);
        }
    }


    public function checkPaymentStatus(Request $request)
    {
        $status = $this->paynow->pollTransaction($request->poll_url);

        return response()->json([
            'status' => $status->paid() ? 'paid' : 'unpaid'
        ]);
    }
}

```

## Json data sample
### initiate-mobile
```bash
{
  "reference": "Invoice 1253",
  "email": authemail, //from Docs
  "phone": "0771111111",
  "method": "ecocash",
  "items": [
    { "name": "Test Item", "price": 1.00 }
  ]
}

```

### Check status
```bash
{
   "poll_url": "from initiate"
}
```
