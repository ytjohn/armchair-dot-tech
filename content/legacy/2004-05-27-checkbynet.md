---

title: CheckByNet
author: ytjohn
date: 2004-05-27 00:05:11

layout: post
permalink: https://www.yourtech.us/2004/checkbynet

---
/*
echeck.class, Copyright 2004 John Hogenmiller john@hsource.net
http://h.hsource.net/
This is an api to interface with ChecksByNet (http://www.checksbynet.com/), provided
by CrossCheck, Inc. This program is in no way (currently) endorsed by Crosscheck, Inc., nor is the author in any way affiliated with CrossCheck, Inc.
Distribution:
This program is released under the GNU Lesser General Public License.
(http://www.opensource.org/licenses/lgpl-license.php)
CONFIGURATION:
Go to the function post_echeck and change the $payto and $paytoid variables
to match what you have with ChecksByNet.  Alternatively, you could pass these
in from the parent script.  If ChecksByNet ever changes their
submission url, you'll have to change the $url variable.
EXAMPLE:
Note that in a production environment, you wouldn't want to pass $<em>REQUEST in
to the post_echeck function as the end user would be able to change the amount,
payto, and paytoid at will.  Ideally, you pull the cbn</em> variables out of
$<em>REQUEST, do any regex checks you want to do on your side, and directly
set the amount from the server side.
post_echeck works by returning an array with the keys "approved" and "err".
$array['approved']   // If set, echeck was approved.
$array['err']        // Human readable text of all errors.
$array['RSPxxxx']    // Server generated error as key, human readable error as value.
Useful for parent scripts that want to do more with the error.
$ec = new echeck;
$result = $ec->post_echeck($_REQUEST);
if ($result[approved])
{ print "Echeck approved
\n"; }
else
{ print "Declined:
\n";
print $result[err];
}
VARIABLES:
Put all the variables into an array and pass that array to post_echeck.
Prefix all variables in the array with cbn</em>
Example:
$checkno = 123;
$check['cbn_checknbr'] = $checkno;
$result = $ec->post_echeck($check);
Checksbynet requires the following variables FROM THE CUSTOMER:
Check Number Value: checknbr            Needs to be greater than 99 / 6 Max
Customer's First Name: writerfirst      15 Max    (only for personal checks)
Customer's Last Name:   writerlast      29 Max
Customer's Business Name: writername    50 Max    (only for business checks)
Customer's Address: writeraddr          Street address required / 50 Max
City: writercity                        30 Max
State: writerst                         2 characters required
Zip: writerzip                          5 digits required
Bank Name: bankname                     50 Max
Bank City: bankcity                     30 Max
Bank State: bankst                      2 characters required
Bank Zip: bankzip                       5 digits required or can be blank
MICR: micr                              80 Max
Customer's Driver's License: idnbr      Do not include dashes or spaces / 40 Max
Driver's License State: idst            2 characters required
Customer's Phone Number: phone          10 digits required / 14 max
Customer's Email: email                 "@" and "." required / 50 Max
They also require these variables FROM THE MERCHANT:
Check Dollar Amount Value: checkamt Needs to be greater than $1.00/ 10 Max characters
These variables are required by the MERCHANT, but can be set in the post_echeck
function, and do not need to be passed in from the parent program.:
Pay to: payto               Who the check is being made out to. 50 Max
Pay to id: paytoid          Merchant id number
*/
class echeck
{
function post_echeck($check)
{
$url = 'https://cross.checksbynet.com/response.asp';
if ($check[cbn_payto])
{ $payto = urlencode($check[cbn_payto]); }
else { $payto = urlencode("Valued CrossCheck Merchant"); }
if ($check[cbn_paytoid])
{ $paytoid = urlencode($check[cbn_paytoid]); }
else { $paytoid = "12345"; }
// For initial testing, you may want to NULL out the paytoid
$paytoid = 0;
$params =
"payto=$payto"
. "&amp;checknbr=".     urlencode($check['cbn_checknbr'])
. "&amp;checkamt=".     urlencode($check['cbn_checkamt'])
. "&amp;writeraddr=".   urlencode($check['cbn_writeraddr'])
. "&amp;writercity=".   urlencode($check['cbn_writercity'])
. "&amp;writerst=".     urlencode($check['cbn_writerst'])
. "&amp;writerzip=".    urlencode($check['cbn_writerzip'])
. "&amp;bankname=".     urlencode($check['cbn_bankname'])
. "&amp;bankcity=".     urlencode($check['cbn_bankcity'])
. "&amp;bankst=".       urlencode($check['cbn_bankst'])
. "&amp;micr=".     urlencode($check['cbn_micr'])
. "&amp;idnbr=".        urlencode($check['cbn_idnbr'])
. "&amp;idst=".     urlencode($check['cbn_idst'])
. "&amp;phone=".        urlencode($check['cbn_phone'])
. "&amp;email=".        urlencode($check['cbn_email']);
if ($check['cbn_bankzip'])     {    $params .=  "&amp;bankzip=".        $check['cbn_bankzip']; }
if ($check['cbn_writername']) { $params .=  "&amp;writername=". $check['cbn_writername'];      }
else { $params .= "&amp;writerfirst=". $check['cbn_writerfirst'] . "&amp;writerlast=". $check['cbn_writerlast']; }
if ($paytoid) { $params .= "&amp;paytoid=". $paytoid; }
$ch = curl_init();
curl_setopt($ch, CURLOPT_POST,1);
curl_setopt($ch, CURLOPT_POSTFIELDS,$params);
curl_setopt($ch, CURLOPT_URL,$url);
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST,  2);
// curl_setopt($ch, CURLOPT_USERAGENT, $defined_vars['HTTP_USER_AGENT']);
curl_setopt($ch, CURLOPT_RETURNTRANSFER,1);
$return=curl_exec($ch);
curl_close ($ch);
$goodresults = $this->quotesplit($return);
foreach ($goodresults as $key => $value)
{
$result[$value] = "ECHECK: Unknown error"; // All known return codes will replace this.
if ($value == 'RSP0051') { $result[err] .= "ECHECK: Configuration error: invalid payto/paytoid";
$result[$value] .= "ECHECK: Configuration error: invalid payto/paytoid"; }
if ($value == 'RSP1101') { $result[err] .= "ECHECK: Bad/blank  check number
\n";
$result[$value] = "ECHECK: Bad/blank  check number
\n"; }
if ($value == 'RSP1102') { $result[err] .= "ECHECK: Bad/blank  check dollar amount
\n";
$result[$value] = "ECHECK: Bad/blank  check dollar amount
\n"; }
if ($value == 'RSP1201') { $result[err] .= "ECHECK: Bad/blank  entry for customer name
\n";
$result[$value] = "ECHECK: Bad/blank  entry for customer name
\n"; }
if ($value == 'RSP1202') { $result[err] .= "ECHECK: Bad/blank  address for customer $check[cbn_writeraddr]
\n";
$result[$value] = "ECHECK: Bad/blank address for customer $check[cbn_writeraddr]
\n"; }
if ($value == 'RSP1203') { $result[err] .= "ECHECK: Bad/blank  city for customer
\n";
$result[$value] = "ECHECK: Bad/blank  city for customer
\n"; }
if ($value == 'RSP1204') { $result[err] .= "ECHECK: Bad/blank  state for customer
\n";
$result[$value] = "ECHECK: Bad/blank  state for customer
\n"; }
if ($value == 'RSP1205') { $result[err] .= "ECHECK: Bad/blank  zip code for customer
\n";
$result[$value] = "ECHECK: Bad/blank  zip code for customer
\n"; }
if ($value == 'RSP1301') { $result[err] .= "ECHECK: Bad/blank  bank name
\n";
$result[$value] = "ECHECK: Bad/blank  bank name
\n"; }
if ($value == 'RSP1302') { $result[err] .= "ECHECK: Bad/blank  city for bank
\n";
$result[$value] = "ECHECK: Bad/blank  city for bank
\n"; }
if ($value == 'RSP1303') { $result[err] .= "ECHECK: Bad/blank  state for bank
\n";
$result[$value] = "ECHECK: Bad/blank  state for bank
\n"; }
if ($value == 'RSP1304') { $result[err] .= "ECHECK: Bad/blank  zip code for bank
\n";
$result[$value] = "ECHECK: Bad/blank  zip code for bank
\n"; }
if ($value == 'RSP1311') { $result[err] .= "ECHECK: Bad/blank  account number
\n";
$result[$value] = "ECHECK: Bad/blank  account number
\n"; }
if ($value == 'RSP1312') { $result[err] .= "ECHECK: Bad/blank  routing number
\n";
$result[$value] = "ECHECK: Bad/blank  routing number
\n"; }
if ($value == 'RSP1313') { $result[err] .= "ECHECK: Bad/blank  micr number
\n";
$result[$value] = "ECHECK: Bad/blank  micr number
\n"; }
if ($value == 'RSP1401') { $result[err] .= "ECHECK: Bad/blank  driver's license number
\n";
$result[$value] = "ECHECK: Bad/blank  driver's license number
\n"; }
if ($value == 'RSP1402') { $result[err] .= "ECHECK: Bad/blank  state for driver's license
\n";
$result[$value] = "ECHECK: Bad/blank  state for driver's license
\n"; }
if ($value == 'RSP1501') { $result[err] .= "ECHECK: Bad/blank  phone number for customer
\n";
$result[$value] = "ECHECK: Bad/blank  phone number for customer
\n"; }
if ($value == 'RSP1502') { $result[err] .= "ECHECK: Bad/blank  email address for customer
\n";
$result[$value] = "ECHECK: Bad/blank  email address for customer
\n"; }
if ($value == 'RSP0000') { $result[approved] = 1; $result[err] = 0; }
if ($value == 'RSP0001') { $result[approved] = 0; $result[err] .= "ECHECK: Declined
\n";
$result[$value] = "ECHECK: ECHECK: Declined
\n"; }
if ($value == 'RSP0020') { $result[err] .= "ECHECK: Check duplication error. We have approved this same check number from the same account in recent history.
\n";
$result[$value] = "ECHECK: Check duplication error. We have approved this same check number from the same account in recent history.
\n"; }
if ($value == 'RSP9999') { $result[err] .= "ECHECK: Unable to process checks at this time
\n";
$result[$value] = "ECHECK: Unable to process checks at this time
\n"; }
// Enable the below line for testing
if ($value == 'RSP0010') { $result[approved] = 1; $result[err] = 0;
$result[$value] = "ECHECK: Test completed.";}
}
if (!$result[err] &amp;&amp; !$result[approved]) { $result[err] .= "ECHECK: Undefined error, not approved"; }
return($result);
} #end function post_echeck
function quotesplit($s)
{
$r = Array();
$p = 0;
$l = strlen($s);
while ($p &lt; $l) {
while (($p &lt; $l) &amp;&amp; (strpos(" \r\t\n",$s[$p]) !== false)) $p++;
if ($s[$p] == '"') {
$p++;
$q = $p;
while (($p &lt; $l) &amp;&amp; ($s[$p] != '"')) {
if ($s[$p] == '&#92;') { $p+=2; continue; }
$p++;
}
$r[] = stripslashes(substr($s, $q, $p-$q));
$p++;
while (($p &lt; $l) &amp;&amp; (strpos(" \r\t\n",$s[$p]) !== false)) $p++;
$p++;
} else if ($s[$p] == "'") {
$p++;
$q = $p;
while (($p &lt; $l) &amp;&amp; ($s[$p] != "'")) {
if ($s[$p] == '&#92;') { $p+=2; continue; }
$p++;
}
$r[] = stripslashes(substr($s, $q, $p-$q));
$p++;
while (($p &lt; $l) &amp;&amp; (strpos(" \r\t\n",$s[$p]) !== false)) $p++;
$p++;
} else {
$q = $p;
while (($p &lt; $l) &amp;&amp; (strpos(",;",$s[$p]) === false)) {
$p++;
}
$r[] = stripslashes(trim(substr($s, $q, $p-$q)));
while (($p &lt; $l) &amp;&amp; (strpos(" \r\t\n",$s[$p]) !== false)) $p++;
$p++;
}
}
return $r;
} # end function quotesplit
} # end class echeck
?>
