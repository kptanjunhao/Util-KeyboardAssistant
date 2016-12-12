# Util-KeyboardAssistant
An auto-adjust keyboard and your control likes some button position util.一个自动适应你的键盘弹起收回以及你的控件位置的工具


    //
    //  KeyboardManager.swift
    //  elcs
    //
    //  Created by 谭钧豪 on 2016/12/12.
    //  Copyright © 2016年 wodeer. All rights reserved.
    //

    import UIKit

    protocol KeyboardChangedDelegate: class {
        /**
         键盘frame变化时调用
         - parameter frame          : 键盘变化后的frame.
         - parameter animateDuration: 键盘变化动画的持续时间.
         - parameter animateCurve   : 键盘变化动画的动画效果.
         */
        func keyboardChanged(frame: CGRect, animateDuration: Double, animateCurve: UIViewAnimationOptions)
    }

    ///监听键盘弹起
    class KeyboardManager{

        weak var delegate: KeyboardChangedDelegate?
        var noShelter: ((_ frame: CGRect, _ animateDuration: Double, _ animateCurve: UIViewAnimationOptions) -> Void)?

        weak var toMoveView: UIView?
        var toMoveViewOriginFrame: CGRect?
        weak var noShelterView: UIView?

        ///代理返回键盘高度的初始化方式
        /**
        func keyboardChanged(frame: CGRect, animateDuration: Double, animateCurve: UIViewAnimationOptions){
           <#code#>
        }
        */
        /// - parameter target     : KeyboardChangedDelegate
        init(_ target: KeyboardChangedDelegate?){
            self.delegate = target
            NotificationCenter.default.addObserver(self, selector: #selector(self.keyboardChanged(sender:)), name: .UIKeyboardWillChangeFrame, object: nil)
        }

        /**
         自动移动不被遮挡控件的初始化方式
         - parameter toMoveView     : 将要被移动的视图.
         - parameter noShelterView  : 不会被键盘遮挡的视图.
         */
        init(toMoveView: UIView, noShelterView: UIView!){
            self.toMoveView = toMoveView
            self.toMoveViewOriginFrame = toMoveView.frame
            self.noShelterView = noShelterView
            NotificationCenter.default.addObserver(self, selector: #selector(self.keyboardChanged(sender:)), name: .UIKeyboardWillChangeFrame, object: nil)
            noShelter = {
                (frame, animateDuration, animateCurve) in
                UIView.animate(withDuration: animateDuration, delay: 0, options: animateCurve, animations: {
                    //键盘在屏幕中的位置的判断
                    if screen.height - frame.origin.y <= 0{
                        //收起则还原控件位置
                        self.toMoveView!.top = self.toMoveViewOriginFrame!.origin.y
                    }else{//弹起则计算偏移，不被遮挡
                        ///控件在屏幕KeyWindow的位置
                        let noShelterRectInWindow = self.noShelterView!.convert(self.noShelterView!.bounds, to: UIApplication.shared.keyWindow!)
                        ///计算控件顶部与键盘顶部的差，加上控件的高度，得出偏移量
                        let offsetY = noShelterRectInWindow.origin.y - frame.origin.y + self.noShelterView!.height
                        self.toMoveView!.top = self.toMoveView!.top - offsetY
                    }
                })
            }
        }

        deinit {
            NotificationCenter.default.removeObserver(self as Any, name: .UIKeyboardWillChangeFrame, object: nil)
        }

        @objc func keyboardChanged(sender: Notification){
            let info = sender.userInfo!
            let frame = (info[UIKeyboardFrameEndUserInfoKey] as AnyObject).cgRectValue ?? CGRect.zero
            let duration = (info[UIKeyboardAnimationDurationUserInfoKey]! as AnyObject).doubleValue ?? 0.3
            let curve = (info[UIKeyboardAnimationCurveUserInfoKey]! as AnyObject).uintValue ?? 7
            let animateCurve = UIViewAnimationOptions(rawValue: curve)
            delegate?.keyboardChanged(frame: frame, animateDuration: duration, animateCurve: animateCurve)
            noShelter?(frame, duration, animateCurve)
        }

    }
