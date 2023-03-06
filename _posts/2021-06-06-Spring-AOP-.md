---
title: Spring Intercepter를 이용한 사용자,개인정보 로그 처리
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
popular: true
categories:
  - Java
  - Spring
  - Intercepter
toc: true
toc_sticky: true
toc_label: 목차
description: Spring Intercepter를 이용한 사용자,개인정보 로그 처리
article_tag1: Spring
article_tag2:
article_tag3:
article_section: Spring Intercepter를 이용한 사용자,개인정보 로그 처리
meta_keywords: Java, Spring, Intercepter
last_modified_at: 2021-06-07T00:00:00+08:00
---

## 개념

인터셉터를 만들려면 HandlerInterceptorAdaptor 클래스를 상속받아야 하고 다음 3개의 메서드 overlade 가능.

- perHandle() :컨트롤러가 호출 전에 실행 메서드
- postHandle : 컨트롤러가 호출 후 실행 메서드
- afterHandel() : 컨트롤러의 처리가 끝나고 화면처리까지 모두 완료시 실행 메서드.

**Tip:** Alt+Shift+S 누르면 Override/Implements Methods... 가능
{: .notice--info}

## dispatcher-servlet.xml 에 설정 추가

```xml
<mvc:interceptors>
	<mvc:interceptor>
		<!-- 로그인 해야지 보이는 URL -->
		<mvc:mapping path="/admin/*.do"/> <!-- 절대경로(/)부터 시작하면 중간경로가 있을 수 있기에 (**)으로 표시 -->
		<mvc:mapping path="/*.do"/> <!-- 절대경로(/)부터 시작하면 중간경로가 있을 수 있기에 (**)으로 표시 -->
		<!-- 로그인 안해도 보이는 URL -->
		<mvc:exclude-mapping path="/login.do"/>
		<mvc:exclude-mapping path="/logoutCheck.do"/>
		<mvc:exclude-mapping path="/loginAgency.do"/>
		<mvc:exclude-mapping path="/loginCheck.do"/>
		<mvc:exclude-mapping path="/setTime.do"/>
		<mvc:exclude-mapping path="/regUserView.do"/>
		<mvc:exclude-mapping path="/insertUser.do"/>
		<mvc:exclude-mapping path="/idCheck.do"/>
		<mvc:exclude-mapping path="/findIdView.do"/>
		<mvc:exclude-mapping path="/getUserId.do"/>
		<bean class="sample.common.serivce.AuthInterceptor" />
	</mvc:interceptor>
</mvc:interceptors>
```

## Intercepter 생성

```java
package sample.common.serivce;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.regex.PatternSyntaxException;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.ModelAndViewDefiningException;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import sample.user.service.UserLogService;
import sample.user.service.UserService;
import sample.user.service.UserVO;

/**
 * 권한 Intercepter 처리
 * @author 저장소
 *
 */
public class AuthInterceptor extends HandlerInterceptorAdapter {

	private static final Logger LOGGER = LoggerFactory.getLogger(AuthInterceptor.class);

	/** User Service */
	@Resource(name = "userService")
	private UserService userService;

	/** User Log Service */
	@Resource(name = "userLogService")
	private UserLogService userLogService;

    /**
     * preHandle
     *
     * 세션에 계정정보(UserVO)가 있는지 여부로 인증 여부를 체크.
     * 계정정보(UserVO)가 없다면, 로그인 페이지로 이동.
     * @throws CommException
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ModelAndViewDefiningException, CommException {

        try {

        	// 로그인시 쿠키에 저장된 세션키 확인
    		CookieUtils cookieUtil = new CookieUtils(request);
    		String secretKey = cookieUtil.getValue("secretKey"); // (로그인시 저장된) 쿠키내 세션키 확인

    		if (secretKey != null && !secretKey.isEmpty()) {

    			UserVO userSessionVo = new UserVO();
    			userSessionVo.setSecretKey(secretKey); // 세션 키
    			UserVO result = userService.selectUserSession(userSessionVo); // 사용자 세션 확인

    			if (result != null) {

    				if ("Y".equals(result.getSessionAliveYn())) { // 현재 세션 유지 상태 일 경우,

    					// 테스트
    					//HttpSession httpSessiontest = request.getSession(true);
    					//httpSessiontest.invalidate();

						HttpSession httpSession = request.getSession(true);

    					// 만약, 현재 내부 Session이 만료 되어 사라졌다면 다시 생성.
    					if (request.getSession().getAttribute("userId") == null) {
    						// 그룹코드 확인
    						String grpCd = result.getGrpCd();

    						if ("T".equals(grpCd)) {
    							grpCd = "TRADER";
    						} else if ("C".equals(grpCd)) {
    							grpCd = "CUSTOMER";
    						} else if ("B".equals(grpCd)) {
    							grpCd = "BUYER";
    						} else if("A".equals(grpCd)) {
    							grpCd = "AGENCY";
    						} else if("I".equals(grpCd)) {
    							grpCd = "INTERPRETER";
    						}

    						// 세션 재등록
    						httpSession.setAttribute("userId", result.getUserId());
    						httpSession.setAttribute("grpCd", grpCd);
							httpSession.setAttribute("userNm", "");
							httpSession.setAttribute("userNmEng", ""); // 사용자명(영문)
							httpSession.setAttribute("deptCd", ""); // 부서코드
							httpSession.setAttribute("ftrdctrYn", ""); // 무역관 여부

							// ===== START OF : 사용자 이름, 부서정보 확인
							UserVO userVo = new UserVO();
							String userId = result.getUserId();
    						userVo.setUserId(userId);
    						UserVO deptInfo = userService.selectUserDeptInfo(userVo);
    						if (deptInfo != null) {
    							String userNm = deptInfo.getUserNm();
    							if ("TRADER".equals(grpCd)) { // 무역관
    								httpSession.setAttribute("userNm", userNm);
    								httpSession.setAttribute("userNmEng", deptInfo.getUserNmEng()); // 사용자명(영문)
    								httpSession.setAttribute("deptCd", deptInfo.getDeptCd()); // 부서코드
    								httpSession.setAttribute("ftrdctrYn", deptInfo.getFtrdctrYn()); // 무역관 여부
    							} else { // 대행사, 통역사
    								httpSession.setAttribute("userNm", userNm);
    								httpSession.setAttribute("userNmEng", userNm); // 사용자명(영문)
    								httpSession.setAttribute("deptCd", ""); // 부서코드
    								httpSession.setAttribute("ftrdctrYn", "N"); // 무역관 여부
    							}
    						}
    					}
    					// ===== END OF : 사용자 이름, 부서정보 확인

    					// ===== START OF : 사용자 권한 확인
    					// Trader만 Admin Page 접근 가능
    					String requestURI = request.getRequestURI().toLowerCase();
    					if (requestURI.contains("/admin/") || requestURI.contains("/admin.do")) {
    						if (!"TRADER".equals(httpSession.getAttribute("grpCd"))) {
    	    					// 로그 아웃 처리
    	    					ModelAndView modelAndView = new ModelAndView("redirect:/logoutCheck.do");
    	    	                throw new ModelAndViewDefiningException(modelAndView);
    						}
    					}
    					// ===== END OF : 사용자 권한 확인

    				} else { // 세션 만료 상태 일 경우,

    					// 로그 아웃 처리
    					ModelAndView modelAndView = new ModelAndView("redirect:/logoutCheck.do");
    	                throw new ModelAndViewDefiningException(modelAndView);
    				}

    			} else { // 세션정보 미존재시,

    				// 로그 아웃 처리
					ModelAndView modelAndView = new ModelAndView("redirect:/logoutCheck.do");
	                throw new ModelAndViewDefiningException(modelAndView);

    			}

    		} else {

                ModelAndView modelAndView = new ModelAndView("redirect:/login.do");
                throw new ModelAndViewDefiningException(modelAndView);

    		}

        } catch (ModelAndViewDefiningException | CommException e ) {

            //modelAndView.addObject("message", "세션이 만료되어 로그아웃 되었습니다. 다시 로그인 해주세요.");
            //ModelAndView modelAndView = new ModelAndView("redirect:/login.do");
            //throw new ModelAndViewDefiningException(modelAndView);
            //return false;
			ModelAndView modelAndView = new ModelAndView("redirect:/logoutCheck.do");
            throw new ModelAndViewDefiningException(modelAndView);

        }

		return true;
    }

    /**
     * 사용자 접속, 개인정보노출 로그 기록
     */
    //@SuppressWarnings("unchecked")
	@Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws ModelAndViewDefiningException, CommException {

    	// 사용자 접속, 개인정보노출 로그 기록
    	try {
    		// === START OF : 사용자 접속 기록 ===
    		Map<String, String> map = new HashMap<String,String>();
	    	HandlerMethod method = (HandlerMethod) handler;
	    	String name = method.getMethod().getName().toLowerCase();

	    	// method의 이름으로 처리형태 구분
	    	if (name.contains("view") || name.contains("detail") || name.contains("search") || name.contains("list")) {
	    		map.put("action", "R"); // 조회
	    	} else if (name.contains("update")) {
	    		map.put("action", "U"); // 수정
	    	} else if (name.contains("insert") || name.contains("add") || name.contains("reg")) {
	    		map.put("action", "C"); // 등록
	    	} else if (name.contains("del") || name.contains("delete")) {
	    		map.put("action", "D"); // 삭제
	    	} else if (name.contains("down") || name.contains("exceldown")) {
	    		map.put("action", "L"); // Download
	    	} else if (name.contains("upload") || name.contains("excelupload")) {
	    		map.put("action", "F"); // Upload
	    	} else {
	    		map.put("action", "R"); // 조회
	    	}

	    	map.put("userId", request.getSession().getAttribute("userId").toString());
	    	map.put("url", request.getRequestURI());
	    	map.put("ip", request.getRemoteAddr());
	    	long userLogkey = userLogService.insertUserAccessLog(map); // 사용자 접속로그 기록
	    	// === END OF : 사용자 접속 기록 ===

	    	// === START OF : 개인정보 노출 기록 ===
	    	if (modelAndView != null) {
		    	Map<String, Object> mvMap = modelAndView.getModel();
		    	Map<String, String> pvMap = null;
		    	Map<String, String> piMap = null;
		        for (String key : mvMap.keySet()){
		            if ("privacyValue".equals(key)) { // 사용자 개인정보 확인 (값 전달 방식이니 바로 등록)
		            	pvMap = this.getPrivacy(mvMap.get(key).toString());
						userLogService.insertUserPrivacyLog(userLogkey, pvMap); // 사용자 개인정보 확인 기록
						continue;
		            }
		            if ("privacyItem".equals(key)) { // 사용자 개인정보 확인 키 확인 (필드명 전달 방식이니 필드를 찾고 값 확인)
		        		piMap = this.getPrivacy(mvMap.get(key).toString()); // 개인정보가 들어간 필드명 확인
		    	        if (piMap != null) { // 개인정보 노출 필드 존재시..

		    	        	for (String key2 : mvMap.keySet()){
		    		            pvMap = null;
		    		            if (mvMap.get(key2) instanceof Map || mvMap.get(key2) instanceof HashMap) {
		    		            	pvMap = this.getPrivacyValue(piMap, (Map<String,?>)mvMap.get(key2));
		    		            } else if (mvMap.get(key2) instanceof List) {
		    		            	pvMap = this.getPrivacyValue(piMap, (List)mvMap.get(key2));
		    		            }
		    		            if (pvMap != null && pvMap.size() > 0) { // 개인정보노출 정보 존재시, insert
		    		            	userLogService.insertUserPrivacyLog(userLogkey, pvMap); // 사용자 개인정보 확인 기록
		    		            }
		    	        	}
		    	        	continue;
		    	        }

		            }
		        }
	    	}
	        // === END OF : 개인정보 노출 기록 ===

	    } finally {
	    }

    }


    /**
     * postHandle
     *
     * 사용자 접속, 개인정보노출 로그 기록
     */
    //@SuppressWarnings("unchecked")
	@Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws ModelAndViewDefiningException, CommException {

    	// 사용자 접속, 개인정보노출 로그 기록
    	try {
    		// === START OF : 사용자 접속 기록 ===
    		Map<String, String> map = new HashMap<String,String>();
	    	HandlerMethod method = (HandlerMethod) handler;
	    	String name = method.getMethod().getName().toLowerCase();

	    	// method의 이름으로 처리형태 구분
	    	if (name.contains("view") || name.contains("detail") || name.contains("search") || name.contains("list")) {
	    		map.put("action", "R"); // 조회
	    	} else if (name.contains("update")) {
	    		map.put("action", "U"); // 수정
	    	} else if (name.contains("insert") || name.contains("add") || name.contains("reg")) {
	    		map.put("action", "C"); // 등록
	    	} else if (name.contains("del") || name.contains("delete")) {
	    		map.put("action", "D"); // 삭제
	    	} else if (name.contains("down") || name.contains("exceldown")) {
	    		map.put("action", "L"); // Download
	    	} else if (name.contains("upload") || name.contains("excelupload")) {
	    		map.put("action", "F"); // Upload
	    	} else {
	    		map.put("action", "R"); // 조회
	    	}

	    	map.put("userId", request.getSession().getAttribute("userId").toString());
	    	map.put("url", request.getRequestURI());
	    	map.put("ip", this.getClientIP(request));
	    	long userLogkey = userLogService.insertUserAccessLog(map); // 사용자 접속로그 기록
	    	// === END OF : 사용자 접속 기록 ===

	    	// === START OF : 개인정보 노출 기록 ===
	    	if (modelAndView != null) {
		    	Map<String, Object> mvMap = modelAndView.getModel();
		    	Map<String, String> pvMap = null;
		    	Map<String, String> piMap = null;
		        for (String key : mvMap.keySet()){
		            if ("privacyValue".equals(key)) { // 사용자 개인정보 확인 (값 전달 방식이니 바로 등록)
		            	pvMap = this.getPrivacy(mvMap.get(key).toString());
				        userLogService.insertUserPrivacyLog(userLogkey, pvMap); // 사용자 개인정보 확인 기록
				        continue;
		            }
		            if ("privacyItem".equals(key)) { // 사용자 개인정보 확인 키 확인 (필드명 전달 방식이니 필드를 찾고 값 확인)
		        		piMap = this.getPrivacy(mvMap.get(key).toString()); // 개인정보가 들어간 필드명 확인
		    	        if (piMap != null) { // 개인정보 노출 필드 존재시..

		    	        	for (String key2 : mvMap.keySet()){
		    		            pvMap = null;
		    		            if (mvMap.get(key2) instanceof Map || mvMap.get(key2) instanceof HashMap) {
		    		            	pvMap = this.getPrivacyValue(piMap, (Map<String,?>)mvMap.get(key2));
		    		            } else if (mvMap.get(key2) instanceof List) {
		    			          	pvMap = this.getPrivacyValue(piMap, (List)mvMap.get(key2));
		    		            }
		    		            if (pvMap != null && pvMap.size() > 0) { // 개인정보노출 정보 존재시, insert
				    		        userLogService.insertUserPrivacyLog(userLogkey, pvMap); // 사용자 개인정보 확인 기록
		    		            }
		    	        	}
		    	        	continue;
		    	        }

		            }
		        }
	    	}
	        // === END OF : 개인정보 노출 기록 ===

	    } finally {
	    }
    }


	/**
     * Privacy 항목을 HashMap으로 변환
     * @param piStr
     * @return
     * @throws CommException
     */
    private Map<String,String> getPrivacy(String piStr) throws CommException {
    	Map<String,String> piMap = null;
    	String[] tmp = null;
    	String[] tmp2 = null;
    	try {
    		if (piStr != null) {
//				piMap = Arrays.asList(piStr.split(",")).stream().collect(
//				Collectors.toMap(e -> (e.contains(":") ? e.split(":")[1] : ""), e -> e.split(":")[0]));
//				piMap.remove("");
    			piMap = new HashMap<String,String>();
	    		tmp = piStr.split(",");

	    		for (String str : tmp) {
	    			if (str.contains(":") && !str.contains("null") && !str.contains("[")) {
	    				tmp2 = str.split(":");
	    				if (tmp2.length > 1) {
	    					piMap.put(tmp2[1], tmp2[0]);
	    				}
	    			}
	    		}

    		}
    	} catch (PatternSyntaxException e) {
    		throw new CommException(e.getMessage(), "PatternSyntaxException");
    	}
    	return piMap;
    }

    /**
     * Map에서 개인정보 노출 관련 필드 filter후  Map으로 생성
     * @param fMap 필터 Map
     * @param sMap 소스 Map
     * @return
     * @throws CommException
     */
    private Map<String,String> getPrivacyValue(Map<String,String> fMap, Map<String,?> sMap) throws CommException {
    	Map<String,String> rMap = null;
    	try {
    		rMap = new HashMap<String,String>();
        	for (String key : fMap.keySet()){
        		if (sMap.containsKey(key)) {
        			rMap.put(sMap.get(key).toString(), fMap.get(key));
        		}
        	}
		} catch (PatternSyntaxException e) {
			throw new CommException(e.getMessage(), "PatternSyntaxException");
		}
		return rMap;
	}

    /**
     * ArrayList<Map>에서 개인정보 노출 관련 필드 filter후 Map으로 생성
     * @param fMap 필터 Map
     * @param sList 소스 List
     * @return
     * @throws CommException
     */
    private Map<String,String> getPrivacyValue(Map<String,String> fMap, List sList) throws CommException {
    	Map<String,String> rMap = null;
    	Map<String,String> tmpMap = null;
    	try {
    		if (sList != null && sList.size() > 0) {
    			rMap = new HashMap<String,String>();
	    		for (int i = 0; i < sList.size(); i++) {
	    			if (sList.get(i) instanceof Map || sList.get(i) instanceof HashMap) {
	    				tmpMap = this.getPrivacyValue(fMap, (Map<String,?>)sList.get(i));
	    				rMap.putAll(tmpMap); // Merge
	    			}
	    		}
    		}
		} catch (PatternSyntaxException e) {
			throw new CommException(e.getMessage(), "PatternSyntaxException");
		}
		return rMap;
	}

	/**
	 * 클라이언트 IP 확인
	 * @param request
	 * @return
	 */
    private String getClientIP(HttpServletRequest request) {
	    String ip = request.getHeader("X-Forwarded-For");
	    if (EgovStringUtil.isEmpty(ip) || "unknown".equalsIgnoreCase(ip)) {
	        ip = request.getHeader("Proxy-Client-IP");
	    }
	    if (EgovStringUtil.isEmpty(ip) || "unknown".equalsIgnoreCase(ip)) {
	        ip = request.getHeader("WL-Proxy-Client-IP");
	    }
	    if (EgovStringUtil.isEmpty(ip) || "unknown".equalsIgnoreCase(ip)) {
	        ip = request.getHeader("HTTP_CLIENT_IP");
	    }
	    if (EgovStringUtil.isEmpty(ip) || "unknown".equalsIgnoreCase(ip)) {
	        ip = request.getHeader("HTTP_X_FORWARDED_FOR");
	    }
	    if (EgovStringUtil.isEmpty(ip) || "unknown".equalsIgnoreCase(ip)) {
	        ip = request.getRemoteAddr();
	    }
	    return ip;
	}

}
```

## Controller에서 호출

```java
/**
	* Sample
	* @return
	* @throws CommException
	*/
@RequestMapping(value = "/sampleCall.do")
public ModelAndView sampleCall( @RequestParam("seq") String seq,Model model, HttpServletRequest request, HttpServletResponse response) throws CommException{
	ModelAndView mv = new ModelAndView("com_sample_call_details"); // jsp 페이지

	SampleVO sampleVO = UserService.sampleDetail(seq); // 조회

	mv.addObject("sampleVO", sampleVO);

	// 개인정보 노출 기록 (필드명 전달 방식)
	mv.addObject("privacyItem", String.format("0001:%s,0006:%s,0001:%s",
			"coname", // 바이어명
			"reptelno", // 대표전화번호
			"chmnname")); // 바이어담당자명
	// 개인정보 노출 기록 (값 전달방식)
	mv.addObject("privacyValue", String.format("0001:%s,0006:%s,0006:%s",
			sampleVO.getCustName(), // 이름
			sampleVO.getCustTelNo(), // 연락처
			sampleVO.getCustMmgTelNo())); // 담당자연락처

	return mv;
}
```
