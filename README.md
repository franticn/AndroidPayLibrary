# AndroidPayLibrary

## 安卓项目支付依赖框架

##导入说明(import)：

在build.gradle直接引用 :

    dependencies {
        implementation 'com.github.franticn:AndroidPayLibrary:v2.2.3'
    }
    注意在Project的build.gradle中添加如下代码
    allprojects {
        repositories {
            jcenter()
            maven { url 'https://jitpack.io' }
        }



##使用说明(Usage)：

1、支付宝

        AliPayFromServer alipay = new AliPayFromServer.Builder().
                with(PaySelectActivity.this).
                create().
                setOnAliPayListener(new AliPayFromServer.OnAliPayListener() {
                    @Override
                    public void onPaySuccess(String resultInfo) {
                    }

                    @Override
                    public void onPayFailure(String resultInfo) {
                    }

                    @Override
                    public void onPayConfirmimg(String resultInfo) {
                    }
                });
        alipay.send(payInfoServer);

2、微信支付

    public void weChatPay() {
        //1.创建微信支付请求
                WechatPayReq wechatPayReq = new WechatPayReq.Builder()
                        .with(this) //activity实例
                        .setAppId(response.getAppId()) //微信支付AppID
                        .setPartnerId(response.getPartnerId())//微信支付商户号
                        .setPrepayId(response.getPrepayId())//预支付码
                        .setNonceStr(response.getNonceStr())
                        .setPackageValue(response.getPackageStr())
                        .setTimeStamp(response.getTimeStamp())//时间戳
                        .setSign(response.getSign())//签名
                        .create();
        //2.发送微信支付请求
                PayAPI.getInstance().sendPayRequest(wechatPayReq);
        //3.关于微信支付的回调
            在“App包名.wxapi”的package中新建WXPayEntryActivity 如下所示，在onResp()方法中执行回调
    }

            public class WXPayEntryActivity extends Activity implements IWXAPIEventHandler {
                            private static final String TAG = "==============WXPayEntryActivity==============";
                            private IWXAPI api;
                            @Override
                            public void onCreate(Bundle savedInstanceState) {
                                super.onCreate(savedInstanceState);
                                api = WXAPIFactory.createWXAPI(this, Constant.APP_ID);
                                api.handleIntent(getIntent(), this);
                            }

                            @Override
                            protected void onNewIntent(Intent intent) {
                                super.onNewIntent(intent);
                                setIntent(intent);
                                api.handleIntent(intent, this);
                            }

                            @Override
                            public void onReq(BaseReq req) {
                                BaseReq abc = req;
                            }

                            @Override
                            public void onResp(BaseResp resp) {
                                Log.e(TAG, "checkArgs=" + resp.checkArgs());
                                if (resp.getType() == ConstantsAPI.COMMAND_PAY_BY_WX) {
                                        AlertDialog.Builder builder = new AlertDialog.Builder(this);
                                        builder.setTitle("提示");
                                        builder.setMessage("微信支付结果：" + "errCode=" + resp.errCode + "; errStr=" + resp.errStr);
                                        builder.show();
                                }
                            }
                        }



3、银联支付

        //步骤1：app这边将购买的商品信息提交给app后台，app后台接收到购买信息之后，将信息提交给银联后台，
        银联接收到支付信息给app后台返回一个交易流水号(tn)
            ...
        //步骤2：通过银联工具类启动支付插件
            UPPayAssistEx.startPay(MainActivity.this, null, null, tn, mMode);
        //步骤3：调用银联支付后，返回给app支付状态的回调
            protected void onActivityResult(int requestCode, int resultCode, Intent data) {
                // 处理银联手机支付控件返回的支付结果
                if (data == null) {
                    return;
                }
                String msg = "";
                /*
                 * 支付控件返回字符串:success、fail、cancel 分别代表支付成功，支付失败，支付取消
                 */
                String str = data.getExtras().getString("pay_result");
                if (str.equalsIgnoreCase("success")) {
                    // 如果想对结果数据验签，可使用下面这段代码，但建议不验签，直接去商户后台查询交易结果
                    // result_data结构见c）result_data参数说明
                    if (data.hasExtra("result_data")) {
                        String result = data.getExtras().getString("result_data");
                        try {
                            JSONObject resultJson = new JSONObject(result);
                            String sign = resultJson.getString("sign");
                            String dataOrg = resultJson.getString("data");
                            // 此处的verify建议送去商户后台做验签
                            // 如要放在手机端验，则代码必须支持更新证书
                            boolean ret = verify(dataOrg, sign, mMode);
                            if (ret) {
                                // 验签成功，显示支付结果
                                msg = "支付成功！";
                            } else {
                                // 验签失败
                                msg = "支付失败！";
                            }
                        } catch (JSONException e) {
                        }
                    }
                    // 结果result_data为成功时，去商户后台查询一下再展示成功
                    msg = "支付成功！";
                } else if (str.equalsIgnoreCase("fail")) {
                    msg = "支付失败！";
                } else if (str.equalsIgnoreCase("cancel")) {
                    msg = "用户取消了支付";
                }
            }