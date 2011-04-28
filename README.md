# Spree PayPal Website Standard

A [Spree](http://spreecommerce.com) extension to allow payments using PayPal Website Standard.

## Before you read further

This README is in the process of thorough rework to describe the current codebase, design decisions and how to use it. But at the moment some parts are out-of-date. Please read the code of the extension, it's pretty well commented and structured. 

## Old Introduction (outdated!)

Overrides the default Spree checkout process and uses offsite payment processing via PayPal's Website Payment Standard (WPS).  

You'll want to test this using a paypal sandbox account first.  Once you have a business account, you'll want to turn on Instant Payment Notification (IPN).  This is how your application will be notified when a transaction is complete.  Certain transactions aren't completed immediately.  Because of this we use IPN for your application to get notified when the transaction is complete.  IPN means that our application gets an incoming request from Paypal when the transaction goes through.  

You may want to implement your own custom logic by adding 'state_machine' hooks.  Just add these hooks in your site extension (don't change the 'pp_website_standard' extension.) Here's an example of how to add the hooks to your site extension.


    fsm = Order.state_machines['state']  
    fsm.after_transition :to => 'paid', :do => :after_payment
    fsm.after_transition :to => 'pending_payment', :do => :after_pending  
    
    Order.class_eval do  
      def after_payment
        # email user and tell them we received their payment
      end
      
      def after_pending
        # email user and thell them that we are processing their order, etc.
      end
    end


## Installation 

Add to your Spree application Gemfile.

    gem "pp_website_standard", :git => "git://github.com/tomash/spree-pp-website-standard.git"

Run the install rake task to copiy migrations (create payment_notifications table)

    rake pp_website_standard:install

Configure, run, test.


## Configuration

Be sure to configure the following configuration parameters. Preferably put it in initializer like config/initializers/pp_website_standard.rb.

Example:

    Spree::Paypal::Config.set(:account => "foo@example.com") 
    Spree::Paypal::Config.set(:ipn_notify_host => "http://example.com:3000")
    Spree::Paypal::Config.set(:success_url => "http://localhost:3000/paypal/confirm")
    

The following configuration options (keys) can be set:

    :account         # email account of store 
    :ipn_notify_host # host prepared to receive IPN notifications
    :success_url     # url the customer is redirected to after successfully completing payment
    :currency_code   #  
    :sandbox_url     # paypal url in sandbox mode: https://www.sandbox.paypal.com/cgi-bin/webscr
    :paypal_url      # paypal url in production:   https://www.paypal.com/cgi-bin/webscr
    :populate_address (true/false) # pre-populate fields of billing address based on spree order data

The ones that should be given sensible defaults (sandbox_url, paypal_url, currency_code) will get ones. Soon.


## IPN Notes (outdated!)

Real world testing indicates that IPN can be very slow.  If you want to test the IPN piece Paypal now has an IPN tool on their developer site.  Just use the correct URL from the hidden field on your Spree checkout screen.  In the IPN tool, change the transaction type to `cart checkout` and change the `mc_gross` variable to match your order total.


## TODO

* README with complete documentation
* Test suite
* Refunds
