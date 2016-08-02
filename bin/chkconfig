#!/bin/sh

##--------------------------------------------------------
#   -- chkconfig.sh - A chkconfig clone in POSIX shell  --
#   -- Copyright (c) 2014-2015 TsT worldmaster.fr       --
##--------------------------------------------------------

# TOSEE: apt-cache show sysv-rc-conf

# TODO:
# - coder la partie list : inetd et xinetd (xinetd.conf)

# Warning: the xinetd info only read inside xinetd.d/*, not inside the xinetd.conf

# in original chkconfig(.pl) some services are ignored : halt rc reboot single skeleton

set -e

_chkconfig__usage() {
	echo 'Usage:'
	echo '        chkconfig -A|--allservices              (together with -l: show all services)'
#	echo '        chkconfig -t|--terse [names]            (shows the links)'
#	echo '        chkconfig -e|--edit  [names]            (configure services)'
#	echo '        chkconfig -s|--set   [name state]...    (configure services)'
	echo '        chkconfig -l|--list [--deps] [names]    (shows the links)'
#	echo '        chkconfig -c|--check name [state]       (check state)'
#	echo '        chkconfig -a|--add   [names]            (runs insserv)'
#	echo '        chkconfig -d|--del   [names]            (runs insserv -r)'
	echo '        chkconfig -h|--help                     (print usage)'
	echo '        chkconfig -f|--force ...                (call insserv with -f)'
	echo ''
#	echo '        chkconfig [name]             same as chkconfig -t'
	echo '        chkconfig name state...      same as chkconfig -s name state'
	echo '        chkconfig --root=<root> ...  use <root> as the root file system'
	echo '        chkconfig --color=<WHEN>     colorize the output.'
	echo '                                     <WHEN> defaults to "auto" or can be "never" or "always".'
}


_chkconfig__list_target_rcN() {
	local d="$1";shift
	local t="$1";shift
	if [ -d "$d" ]; then
		find "$d/" -type l -name '*'"$t" \
		| (
		while read dir_file; do
			[ "$(dirname "$dir_file")" != "$d" ] && continue
			local name0="$(basename "$dir_file")"

			local act="$(printf %.1c "$name0")"
			[ "$act" != "S" ] && continue

			local name1="${name0#[SK][0-9][0-9]}"
			[ "$name1" != "$t" ] && continue

			if [ -f "${rootdir:-/}${initdir:-etc/init.d}/$name1" ]; then
				return 0
			fi
		done
		return 1
		)
		return $?
	fi
	return 1
}

_chkconfig__known_rc() {
	local target="$1"; shift
	for l in 0 1 2 3 4 5 6 B S; do
		local d
		case "$l" in
			[0-9]|S)  d="${rootdir:-/}etc/rc${l}.d" ;;
			B)        d="${rootdir:-/}etc/boot.d"   ;;
		esac
		[ ! -d "$d" ] && continue
		if [ -f "${rootdir:-/}${initdir:-etc/init.d}/$target" ] || find "$d" -type l -name '[KS][0-9][0-9]'"$target" |grep -q ''; then
			return 0
		fi
	done
	return 1
}

_chkconfig_color() {
	local target="${1:-green}";shift
	local fmt
	if $usecolor; then
		case "$target" in
			green) fmt='\033[32;01m%s\033[0m' ;;
		esac
	fi
	printf "${fmt:-%s}" "$1"
}

_chkconfig__list_target_sysv() {
	local target="$1"; shift

	if ! _chkconfig__known_rc "$target"; then
		echo >&2 "(sysv): $target: unknown service"
		return 1
	fi
	printf "%-24s" "$target"
	for l in "$@"; do
		local d
		case "$l" in
			[0-9]|S) d="${rootdir:-/}etc/rc${l}.d" ;;
			B)       d="${rootdir:-/}etc/boot.d" ;;
		esac
		[ ! -d "$d" ] && continue
		if _chkconfig__list_target_rcN "$d" "$target"; then
			_chkconfig_color green "  $l:on "
		elif [ "$l" != "S" ]; then
			_chkconfig_color none "  $l:off"
		fi
	done
	#if ${printdeps}; then
	#	printf '\t' + print getdeps_rc($s)
	#fi
	printf '\n'
}

_chkconfig__list_target_systemd() {
	local target="$1"; shift
	local targetlong="$1"; shift
	local state="$1"; shift
	if [ -z "$state" ]; then
		#echo >&2 "#debug: targetlong=$targetlong"
		state="$("$SYSTEMCTL" list-unit-files --no-legend --no-pager "$targetlong")"
		state="${state#* }"
	else
		case "$state" in
			('enabled'|'disabled') ;;
			('masked'|'static') return ;;
			('invalid') return ;;
			(*) return
		esac
	fi

	printf "%-24s" "$target"
	for l in 0 1 2 3 4 5 B S; do
		case "$l" in
			("${runlevel:-2}")
				case "$state" in
					enabled)	_chkconfig_color green "  $l:on " ;;
					disabled)	_chkconfig_color none "  $l:off" ;;
					*)		_chkconfig_color none " $l:?  " ;;
				esac
				
#				if systemdctl is-active "${target}.service" >/dev/null; then
#					_chkconfig_color green "  $l:on "
#				else
#					_chkconfig_color none "  $l:off"
#				fi
			;;
			B|S) continue ;;
			(*)
				_chkconfig_color none "  $l:?  "
			;;
		esac
	done
	printf '\n'
}

_chkconfig__list_targets_sysv() {
	local target
	for target in "$@"; do
		case "$target" in
			README) continue ;;
			functions) continue ;;
			.*) continue ;;
		esac
		_chkconfig__list_target_sysv "$target" 0 1 2 3 4 5 B S || true
	done
}


_chkconfig__list_targets_systemd() {
	local runlevel="$(runlevel)"
	case "$runlevel" in
		('N '?) runlevel="${runlevel#N }" ;;
		(*) runlevel="2"
	esac

#	local targets
#	if [ $# -eq 0 ]; then
#		targets="-a"
#	else
#		targets=""
#		local short
#		for short in "$@"; do
#			targets="${targets:+ }${short%.service}.service"
#		done
#	fi
	(
	if [ $# -eq 0 ]; then
		"$SYSTEMCTL" list-unit-files -t service --state=enabled,disabled --no-pager --no-legend
	else
		for short in "$@"; do
			"$SYSTEMCTL" list-unit-files -t service --state=enabled,disabled --no-pager --no-legend ${short%.service}.service
		done
	fi
	) \
	| while read -r targetlong state _ign; do
		local target="$targetlong"
		case "$targetlong" in
			(*'@.'*) continue ;;
			(*'.service') target="${targetlong%.service}" ;;
#			(*) continue ;;
		esac
		_chkconfig__list_target_systemd "$target" "$targetlong" "$state"
	done
}
_chkconfig__list_targets() {
	local SYSTEMCTL
	SYSTEMCTL="$(command -v systemctl)" || true
	if [ $# -eq 0 ]; then
		echo "# sysv:"
		_chkconfig__list_targets_sysv $(_chkconfig__listinitdname)
		if [ -n "$SYSTEMCTL" ];then
			echo "# systemd:"
			_chkconfig__list_targets_systemd
		fi
	else
		echo "# sysv:"
		_chkconfig__list_targets_sysv "$@"
		if [ -n "$SYSTEMCTL" ];then
			echo "# systemd:"
			_chkconfig__list_targets_systemd "$@"
		fi
	fi
}


_chkconfig__listinitdname() {
	find "${rootdir:-/}etc/init.d/" \( -type f -o -type l \) -a -print \
	| sort -u \
	| while read -r d_f; do
		case "$d_f" in
			*/halt|*/rc|*/reboot|*/single|*/skeleton) continue ;;
		esac
		basename "$d_f"
	done
}

_chkconfig__list_xinetd() {
	[ -d "${rootdir:-/}etc/xinetd.d/" ] || return
	echo 'xinetd based services:'
	(
		cd "${rootdir:-/}etc/xinetd.d/" && \
		grep 2>&- -rH 'disable.*=.*\(yes\|no\)' . | tr -d ' \t' | sort -u | (
			IFS=:
			while read -r p s; do
				case "$s" in
					disable=yes)	printf '%8s%-19s %s\n' "" "${p#./}:" "off" ;;
					disable=no)	printf '%8s%-19s %s\n' "" "${p#./}:" "$(_chkconfig_color green "on")"  ;;
					*) echo "Error: unknown case for xinetd: $s"
				esac
			done
		)
		# list service that is not readable (maybe a permission issue)
		cd "${rootdir:-/}etc/xinetd.d/" && \
		for p in *; do
			if [ -f "$p" ] && [ ! -r "$p" ]; then
				printf '%8s%-19s %s\n' "" "${p#./}:" "?  "
			fi
		done
	)
}



_chkconfig__service_enable() {
	local UPDATERC SYSTEMCTL
	_chkconfig__check_for_change

	[ -z "$UPDATERC"  ] || _chkconfig__sysv_service_enable    "$1"
	[ -z "$SYSTEMCTL" ] || _chkconfig__systemd_service_enable "$1"
}
_chkconfig__service_disable() {
	local UPDATERC SYSTEMCTL
	_chkconfig__check_for_change

	[ -z "$UPDATERC"  ] || _chkconfig__sysv_service_disable    "$1"
	[ -z "$SYSTEMCTL" ] || _chkconfig__systemd_service_disable "$1"
}


_chkconfig__sysv_service_enable() {
	"$UPDATERC" ${force:+-f} "$1" enable >/dev/null 2>&1
}

_chkconfig__sysv_service_disable() {
	"$UPDATERC" ${force:+-f} "$1" disable >/dev/null 2>&1
}

_chkconfig__systemd_service_enable() {
	if ! "$SYSTEMCTL" -q is-enabled "${1%.service}.service" >/dev/null; then
		"$SYSTEMCTL" enable "${1%.service}.service" >/dev/null 2>&1
	else
		echo "$1 Already enabled"
	fi
}

_chkconfig__systemd_service_disable() {
	if "$SYSTEMCTL" -q is-enabled "${1%.service}.service"; then
		"$SYSTEMCTL" disable "${1%.service}.service" >/dev/null 2>&1
	else
		echo "$1 Already disabled"
	fi
}


_chkconfig__check_for_change() {
	UPDATERC="$(command -v update-rc.d)" || true
	SYSTEMCTL="$(command -v systemctl)" || true

#	local INSSERV
#	INSSERV="$(command -v insserv)"

	if [ -z "$UPDATERC" ] && [ -z "$SYSTEMCTL" ]; then
		echo >&2 "No such update-rc.d or systemctl"
		return 1
	fi
#	if [ -z "$INSSERV" ]; then
#		echo >&2 "No such inssert"
#		return 1
#	fi
}

_chkconfig__needhelp() {
	while [ $# -gt 0 ]; do
		case "$1" in
			-h|--help) _chkconfig__usage; exit 0 ;;
			--) break ;;
		esac
		shift
	done
}
_chkconfig__needhelp "$@"

chkconfig() {
	local rootdir='/'

	local color="auto"
	local usecolor=true
	[ -t 1 ] || usecolor=false

	local force=""

	local initdir='etc/init.d'
	# local inetddir='etc/inetd.d' ;# not used yet
	# local xinetddir='etc/xinetd.d' ;# not used yet

	local SHOWDEPS=false
	local ALLSERVICES=false
	local DO_LIST=false
	local NOACTION=true

	if [ $# -eq 0 ]; then
		ALLSERVICES=true ;# default action is --allservices
	else

		while [ $# -gt 0 ]; do
			case "$1" in
				-f|--force)
					force=-f
				;;
				--root)
					shift;
					rootdir="${1%/}"
					case "$rootdir" in
						*/) ;;
						*) rootdir="${rootdir}/"
					esac
				;;
				--root=*)
					rootdir="${1#*=}"
					case "$rootdir" in
						*/) ;;
						*) rootdir="${rootdir}/"
					esac
				;;
				--color|--color=*)
					if [ "$1" = "--color" ]; then
						shift
						color="$1"
					else
						color="${1#*=}"
					fi
					case "$color" in
						auto)
							[ -t 1 ] && usecolor=true || usecolor=false
						;;
						never)
							usecolor=false
						;;
						always)
							usecolor=true
						;;
						*) echo >&2 "Invalid color option value"; exit 1
					esac
				;;
				-A|--allservices)
					if [ "${DO_LIST:-false}" = "false" ]; then
						NOACTION=false
						ALLSERVICES=true
					fi
				;;
				-l|--list)
					ALLSERVICES=false
					NOACTION=false
					DO_LIST=true
				;;
				--deps)
					SHOWDEPS=true ;# only used with --list, is not a action.
				;;
				-*)
					echo >&2 "Unknown option $1"
					#_chkconfig__usage
					exit 1
				;;
				*) break
			esac
			shift
		done

		if [ "${NOACTION}" = "true" ]; then
			ALLSERVICES=true
		fi
	fi

	if $DO_LIST; then
		if [ $# -eq 0 ]; then
			# with ou without SHOWDEPS
			_chkconfig__list_targets
			_chkconfig__list_xinetd
			exit $?
		else
			# with ou without SHOWDEPS
			_chkconfig__list_targets "$@"; exit $?
		fi
	fi

	if $NOACTION; then
		while [ $# -gt 0 ]; do
			case "$2" in
				[0-9]*) shift 2 ; continue ;;
				on)
					if _chkconfig__service_enable "$1"; then
						_chkconfig__list_targets "$1"
					else
						echo "Fail to enable '$1'."
					fi
					shift 2
					continue
				;;
				off)
					if _chkconfig__service_disable "$1"; then
						_chkconfig__list_targets "$1"
					else
						echo "Fail to disable '$1'."
					fi
					shift 2
					continue
				;;
				*)
					_chkconfig__list_targets "$1"
					if [ "$1" = "xinetd" ]; then
						_chkconfig__list_xinetd
					fi
			esac
			shift
		done
		ALLSERVICES=false
	fi

	if $ALLSERVICES; then
		#echo "all services not implemented yet"
		_chkconfig__list_targets
	fi
}
chkconfig "$@"
exit $?
